# 📦 WAProto — Fitur Baru (Tag 96–127)
> BaileysMod fork by @TimeKode
> All create by claude ai
> Proto source: WhatsApp Web `2.3000.1042381694`

---

## 🗂️ Kategori Field

### ✅ Custom Fork (BaileysMod only)
> Field yang ditambahin manual di fork ini, **bukan** dari proto resmi WA

| Tag | Field | Class | Status Handler |
|-----|-------|-------|----------------|
| 104 | `groupStatusMessageV3` | `FutureProofMessage` | ✅ `handleGroupStory` |

---

### 📦 FutureProofMessage (Wrapper)
> Semua field ini pakai wrapper `{ message: { ...innerMessage } }`.  
> Render di app resmi tergantung **server-side feature flag** WA — proto bisa dikirim, tapi belum tentu tampil di penerima.

| Tag | Field | Keterangan | Status |
|-----|-------|-----------|--------|
| 37 | `viewOnceMessage` | View once (lihat sekali) | ✅ Native Baileys |
| 40 | `ephemeralMessage` | Pesan hilang otomatis | ✅ Native Baileys |
| 53 | `documentWithCaptionMessage` | Dokumen + caption | ✅ Native Baileys |
| 55 | `viewOnceMessageV2` | View once versi 2 | ✅ Native Baileys |
| 58 | `editedMessage` | Pesan yang diedit | ✅ Native Baileys |
| 59 | `viewOnceMessageV2Extension` | Extension view once v2 | ✅ Native Baileys |
| 62 | `groupMentionedMessage` | Tag/mention grup | ✅ Native Baileys |
| 67 | `botInvokeMessage` | Bot invoke | ⚠️ Belum ada handler |
| 74 | `lottieStickerMessage` | Stiker animasi Lottie | ⚠️ Belum ada handler |
| 85 | `eventCoverImage` | Cover image event | ⚠️ Belum ada handler |
| 87 | `statusMentionMessage` | Mention di status | ⚠️ Belum ada handler |
| 90 | `pollCreationOptionImageMessage` | Gambar di opsi poll | ⚠️ Belum ada handler |
| 91 | `associatedChildMessage` | Pesan anak (album child) | ⚠️ Belum ada handler |
| 92 | `groupStatusMentionMessage` | Mention di group status | ⚠️ Belum ada handler |
| 93 | `pollCreationMessageV4` | Poll v4 | ⚠️ Belum ada handler |
| 95 | `statusAddYours` | "Add yours" di status | ⚠️ Belum ada handler |
| 96 | `groupStatusMessage` | Group status (versi awal) | ✅ `handleGroupStory` |
| 99 | `limitSharingMessage` | Limit sharing | ⚠️ Belum ada handler |
| 100 | `botTaskMessage` | Bot task | ⚠️ Belum ada handler |
| 101 | `questionMessage` | Pertanyaan di status | ⚠️ Belum ada handler |
| 103 | `groupStatusMessageV2` | Group status v2 (green ring) | ✅ `handleGroupStory` |
| 104 | `botForwardedMessage` | Pesan yang di-forward bot | ⚠️ Belum ada handler |
| 106 | `questionReplyMessage` | Reply ke pertanyaan status | ⚠️ Belum ada handler |
| 116 | `newsletterAdminProfileMessage` | Profil admin newsletter | ⚠️ Belum ada handler |
| 117 | `newsletterAdminProfileMessageV2` | Profil admin newsletter v2 | ⚠️ Belum ada handler |
| 118 | `spoilerMessage` | Spoiler/blur media | ✅ via `contextInfo.isSpoiler` |
| 126 | `newsletterAdminProfileStatusMessage` | Status profil admin newsletter | ⚠️ Belum ada handler |

#### Cara pakai FutureProofMessage (manual):
```js
const msg = await sock.generateWAMessageFromContent(jid, {
    viewOnceMessage: {          // ganti nama field sesuai kebutuhan
        message: {
            imageMessage: { ... }  // inner message biasa
        }
    }
}, { userJid: sock.user.id });

await sock.relayMessage(jid, msg.message, { messageId: msg.key.id });
```

#### Cara pakai `spoilerMessage`:
```js
// cukup tambahin spoiler: true, patch di messages.js udah handle sisanya
sock.sendMessage(jid, {
    image: fs.readFileSync('./foto.jpg'),
    caption: 'ini spoiler',
    spoiler: true
});
```

> **Catatan:** `spoilerMessage` (tag 118) di proto tetap ada sebagai wrapper,
> tapi cara yang **bener dan confirmed dari proto resmi** adalah via
> `contextInfo.isSpoiler = true` yang otomatis ke-inject oleh patch `messages.js`.

---

### ✅ Field Baru dengan Class Sendiri (Tag 105–127)
> Field-field ini punya class proto khusus (bukan FutureProofMessage).  
> Semua struct field-nya **confirmed** dari `.proto` resmi WA `2.3000.1042381694`.

---

#### 📨 `statusQuestionAnswerMessage` — Tag 105
Jawab pertanyaan yang nempel di status orang.

**Struktur:**
```
key: MessageKey     — reference ke status yang ada pertanyaannya
text: string        — isi jawaban
```

**Cara pakai:**
```js
sock.sendMessage(jid, {
    statusQuestionAnswerMessage: {
        statusKey: {
            remoteJid: 'status@broadcast',
            fromMe: false,
            id: 'STATUS_ID',          // dari msg.key.id di messages.upsert
            participant: '628xxx@s.whatsapp.net'  // pemilik status
        },
        text: 'Jawaban gw'
    }
});
```

---

#### 💬 `questionResponseMessage` — Tag 107
Response ke pertanyaan (mirip statusQuestionAnswerMessage, tapi untuk pertanyaan biasa).

**Struktur:**
```
key: MessageKey     — reference ke pesan pertanyaan
text: string        — isi jawaban
```

**Cara pakai:**
```js
sock.sendMessage(jid, {
    questionResponseMessage: {
        key: {
            remoteJid: jid,
            id: 'MESSAGE_ID_PERTANYAAN'
        },
        text: 'jawaban gw'
    }
});
```

---

#### 📌 `statusQuotedMessage` — Tag 109
Quote/re-share status orang ke chat.

**Struktur:**
```
type: enum          — QUESTION_ANSWER = 1
text: string        — teks dari status asli
thumbnail: bytes    — preview thumbnail (opsional)
originalStatusId: MessageKey  — ID status asli
```

**Cara pakai:**
```js
sock.sendMessage(jid, {
    statusQuotedMessage: {
        type: 1,  // QUESTION_ANSWER
        text: 'isi teks status asli',
        originalStatusId: {
            remoteJid: 'status@broadcast',
            id: 'STATUS_ID',
            participant: '628xxx@s.whatsapp.net'
        }
    }
});
```

---

#### 🎉 `statusStickerInteractionMessage` — Tag 110
Kirim reaksi stiker ke status orang.

**Struktur:**
```
key: MessageKey     — reference ke status
stickerKey: string  — key/id stiker yang direaksikan
type: enum          — UNKNOWN = 0, REACTION = 1
```

**Cara pakai:**
```js
sock.sendMessage(jid, {
    statusStickerInteractionMessage: {
        key: {
            remoteJid: 'status@broadcast',
            id: 'STATUS_ID',
            participant: '628xxx@s.whatsapp.net'
        },
        stickerKey: 'STICKER_KEY',
        type: 1  // REACTION
    }
});
```

---

#### 📊 `pollCreationMessageV5` — Tag 111
Poll versi 5 (reuse class `PollCreationMessage`).

**Cara pakai:**
```js
sock.sendMessage(jid, {
    poll: {
        name: 'Pertanyaan poll',
        values: ['Opsi 1', 'Opsi 2', 'Opsi 3'],
        selectableCount: 1
    }
});
// atau manual via pollCreationMessageV5
```

---

#### ➕ `pollAddOptionMessage` — Tag 121
Tambahin opsi baru ke poll yang udah ada.

**Struktur:**
```
pollCreationMessageKey: MessageKey  — key ke pesan poll asli
addOption: { optionName: string }   — opsi yang mau ditambahin
metadata: PollUpdateMessageMetadata — metadata (opsional)
```

**Cara pakai:**
```js
sock.sendMessage(jid, {
    pollAddOptionMessage: {
        pollCreationMessageKey: {
            remoteJid: jid,
            id: 'POLL_MESSAGE_ID',
            fromMe: false
        },
        addOption: {
            optionName: 'Opsi Baru'
        }
    }
});
```

---

#### 🎪 `eventInviteMessage` — Tag 122
Kirim undangan event ke chat/grup.

**Struktur:**
```
contextInfo: ContextInfo  — context (opsional)
eventId: string           — ID event unik
eventTitle: string        — judul event
jpegThumbnail: bytes      — thumbnail (opsional)
startTime: int64          — waktu mulai (Unix timestamp ms)
endTime: int64            — waktu selesai (Unix timestamp ms)
caption: string           — deskripsi
isCanceled: bool          — apakah event dibatalkan
callLink: string          — link call/meet (opsional)
```

**Cara pakai:**
```js
sock.sendMessage(jid, {
    eventInviteMessage: {
        eventId: 'EVT_' + Date.now(),
        eventTitle: 'Nama Event',
        caption: 'Deskripsi event disini',
        startTime: Math.floor(Date.now() / 1000),
        endTime: Math.floor(Date.now() / 1000) + 3600,
        callLink: 'https://meet.google.com/xxx'
    }
});
```

---

#### 🔒 `conditionalRevealMessage` — Tag 120
Pesan yang reveal secara kondisional (terjadwal).

**Struktur:**
```
encPayload: bytes                    — payload terenkripsi
encIv: bytes                         — IV enkripsi
conditionalRevealMessageType: enum   — UNKNOWN=0, SCHEDULED_MESSAGE=1
revealKeyId: string                  — ID kunci reveal
```

> ⚠️ Field ini butuh encryption payload yang valid dari server WA. Nggak bisa dibikin manual tanpa key dari server.

---

#### 💸 `paymentReminderMessage` — Tag 124
Pengingat pembayaran.

**Struktur:**
```
reminderId: string      — ID reminder
instanceId: string      — ID instance
description: string     — deskripsi
frequency: enum         — WEEKLY=1, BI_WEEKLY=2, MONTHLY=3, QUARTERLY=4
status: enum            — ACTIVE=1, CANCELLED=2, STOPPED=3, EXPIRED=4, PAID=5
payeeVpa: string        — VPA penerima
payeeJid: string        — JID penerima
payerJid: string        — JID pembayar
amount: Money           — jumlah
```

---

#### 💰 `splitPaymentMessage` — Tag 125
Bayar bareng / split bill.

**Struktur:**
```
splitId: string                          — ID split
totalAmount: Money                       — total jumlah
description: string                      — deskripsi
requesterJid: string                     — JID yang minta
participants: SplitPaymentParticipant[]  — daftar peserta
createdAtMs: int64                       — waktu dibuat (ms)
contextInfo: ContextInfo
```

**Participant:**
```
jid: string                    — JID peserta
amount: Money                  — jumlah tagihan
status: enum                   — PENDING=0, PAID=1
```

---

### 🚫 Sengaja Tidak Diimplementasi

| Tag | Field | Alasan |
|-----|-------|--------|
| 123 | `groupRootKeyShare` | Bagian key-exchange Signal E2E — forge manual bisa ngerusak session enkripsi grup |
| 127 | `rootSecretDistributeMessage` | Sama seperti di atas |

---

## 📡 Cara Ambil STATUS_ID

```js
const statusCache = new Map();

sock.ev.on('messages.upsert', ({ messages }) => {
    for (const msg of messages) {
        if (msg.key.remoteJid === 'status@broadcast') {
            statusCache.set(msg.key.id, {
                id: msg.key.id,
                participant: msg.key.participant,
                message: msg.message
            });
            console.log('STATUS MASUK:', msg.key.id, 'dari', msg.key.participant);
        }
    }
});
```

---

## 🔁 Pattern Umum Kirim Pesan Custom

Semua fitur di atas bisa juga dikirim manual via `relayMessage` tanpa `sendMessage`:

```js
const { generateWAMessageFromContent } = require('@whiskeysockets/baileys');

const msg = await generateWAMessageFromContent(jid, {
    NAMA_FIELD: { ...isi }
}, { userJid: sock.user.id });  // userJid WAJIB diisi!

await sock.relayMessage(jid, msg.message, { messageId: msg.key.id });
```

> **`userJid` wajib diisi `sock.user.id`** — tanpa ini beberapa fitur (green ring, album linking, dll) gagal render di sisi penerima.
