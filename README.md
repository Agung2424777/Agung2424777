# Bot WhatsApp CindyAmelia

Bot WhatsApp all-in-one full fitur!
- Tanpa scan QR code
- Auto reply
- Downloader, converter, auto save media
- Broadcast, game, AI chat, dll.

## Cara Install di Termux

```bash
pkg update && pkg upgrade -y
pkg install git -y
pkg install nodejs -y
git clone https://github.com/Agung2424777/Agung2424777.git CindyAmelia
cd CindyAmelia
npm install
npm start
 kata yang dilarang. Harap berhati-hati!` })
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
