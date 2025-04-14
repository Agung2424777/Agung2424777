const { default: makeWASocket, useSingleFileAuthState, DisconnectReason } = require('@whiskeysockets/baileys')
const fs = require('fs')
const P = require('pino')
const qrcode = require('qrcode-terminal')
const crypto = require('crypto')

// Daftar kata-kata yang dilarang
const forbiddenWords = ['kata1', 'kata2', 'kata3'];  // Ganti dengan kata-kata yang dilarang

// ID Admin, ganti dengan ID WhatsApp admin Anda
const adminIds = ['+1234567890@c.us']; // Ganti dengan nomor WhatsApp admin

let violationCount = {};  // Menyimpan jumlah pelanggaran per pengguna

const { state, saveState } = useSingleFileAuthState('./session.json')

async function startBot() {
    const sock = makeWASocket({
        logger: P({ level: 'silent' }),
        printQRInTerminal: true,
        auth: state
    })

    sock.ev.on('creds.update', saveState)

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect } = update
        if (connection === 'close') {
            const shouldReconnect = (lastDisconnect.error)?.output?.statusCode !== DisconnectReason.loggedOut
            console.log('Connection closed due to ', lastDisconnect.error, ', reconnecting ', shouldReconnect)
            if (shouldReconnect) {
                startBot()
            }
        } else if (connection === 'open') {
            console.log('Bot connected!')
        }
    })

    sock.ev.on('messages.upsert', async ({ messages, type }) => {
        console.log('Received messages: ', messages)

        const msg = messages[0]
        if (!msg.message) return

        const from = msg.key.remoteJid
        const messageContent = msg.message.conversation || msg.message.extendedTextMessage?.text

        if (messageContent) {
            // Cek apakah pesan mengandung kata-kata yang dilarang
            let violation = false
            for (let word of forbiddenWords) {
                if (messageContent.toLowerCase().includes(word)) {
                    violation = true
                    break
                }
            }

            if (violation) {
                if (!violationCount[from]) {
                    violationCount[from] = 0
                }

                violationCount[from]++

                // Kirim peringatan pertama kali
                if (violationCount[from] === 1) {
                    await sock.sendMessage(from, { text: `🚫 Anda melanggar aturan! Menggunakan kata yang dilarang. Harap berhati-hati!` })
                }

                // Jika pelanggaran kedua, kick pengguna
                if (violationCount[from] >= 2) {
                    // Kirim peringatan kedua
                    await sock.sendMessage(from, { text: `⚠️ Peringatan: Anda telah melanggar aturan dua kali. Anda akan dikeluarkan dari grup jika melanggar lagi.` })

                    // Jika admin sudah mengonfirmasi dan pengguna melanggar lagi, kick
                    if (adminIds.includes(from)) {
                        await sock.groupRemove(from, [from])
                        await sock.sendMessage(from, { text: `🚷 Anda telah dikeluarkan karena melanggar aturan!` })
                    }
                }
            } else {
                violationCount[from] = 0; // Reset jika tidak ada pelanggaran
            }

            // Perintah 'menu'
            if (messageContent.toLowerCase() === 'menu') {
                await sock.sendMessage(from, { text: `*Hello! Ini bot agungagung*\n\n> Menu:\n- AI Chat\n- Downloader\n- Converter\n- Broadcast\n\nKetik perintah untuk mulai!` })
            }

            // AI Chat contoh
            if (messageContent.toLowerCase().includes('halo')) {
                await sock.sendMessage(from, { text: `Hai juga! Aku bot agungagung.` })
            }

            // Broadcast contoh
            if (messageContent.toLowerCase() === 'broadcast') {
                let chats = await sock.groupFetchAllParticipating()
                for (let id in chats) {
                    sock.sendMessage(id, { text: `Ini adalah broadcast dari bot agungagung!` })
                }
            }

            // Perkenalan fitur
            if (messageContent.toLowerCase() === 'perkenalkan') {
                await sock.sendMessage(from, { text: `Halo! Aku bot agungagung, siap membantu kamu dengan berbagai fitur!\n\nBerikut adalah beberapa fitur yang tersedia:\n- AI Chat: Berbicara dengan bot.\n- Downloader: Download berbagai file media.\n- Converter: Mengonversi berbagai file.\n- Broadcast: Kirim pesan ke banyak chat.\n\nKetik 'menu' untuk melihat daftar fitur lainnya!` })
            }
        }
    })
}

startBot()
