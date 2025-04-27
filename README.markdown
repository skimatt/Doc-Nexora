### AI Web App: Simple AI Chat Application

Selamat datang di dokumentasi resmi untuk **AI Web App**, sebuah aplikasi web sederhana yang memungkinkan pengguna untuk login, membuat sesi chat, mengirim pesan, dan mendapatkan jawaban dari AI dengan data yang tersimpan di database. Dokumentasi ini dirancang untuk memandu Anda secara rinci melalui proses setup, pengembangan, dan deployment aplikasi ini.

---

## 1. Deskripsi Singkat

**AI Web App** adalah aplikasi berbasis web yang memungkinkan pengguna untuk:

- Login atau mendaftar menggunakan email dan kata sandi.
- Membuat sesi chat baru.
- Mengirim pesan dan menerima respons dari AI.
- Menyimpan semua pesan (dari pengguna dan AI) ke database untuk keperluan histori.

Aplikasi ini dibangun dengan teknologi modern untuk memastikan performa, keamanan, dan kemudahan pengembangan lebih lanjut.

---

## 2. Teknologi yang Digunakan

Berikut adalah daftar teknologi yang digunakan dalam pengembangan aplikasi ini:

| **Tools** | **Keterangan** |
| --- | --- |
| React.js (JavaScript) | Frontend untuk antarmuka pengguna |
| Supabase | Database dan autentikasi pengguna |
| OpenRouter API | Mesin AI untuk menghasilkan respons |
| Cloudflare Worker | Proxy API untuk keamanan |
| Vercel | Hosting aplikasi web |

---

## 3. Arsitektur Aplikasi

Berikut adalah diagram arsitektur aplikasi yang menggambarkan alur kerja dari frontend hingga backend:

```mermaid
graph TD
    A[User Browser] -->|Login/SignUp| B[React.js Frontend]
    B -->|Auth Request| C[Supabase Authentication]
    B -->|Chat Request| D[Supabase Database]
    B -->|AI Request| E[Cloudflare Worker]
    E -->|Proxy Request| F[OpenRouter API]
    C -->|Store User Data| D
    D -->|Store Chat & Messages| B
    F -->|AI Response| E
    E -->|Forward Response| B
```

**Penjelasan Diagram:**

- Pengguna berinteraksi melalui browser dengan frontend React.js.
- Frontend mengelola autentikasi melalui Supabase.
- Pesan pengguna disimpan di database Supabase dan dikirim ke Cloudflare Worker.
- Cloudflare Worker bertindak sebagai proxy untuk mengamankan API key dan mengirimkan permintaan ke OpenRouter API.
- Respons AI dikembalikan ke frontend dan disimpan di database.

---

## 4. Cara Setup Aplikasi (Langkah 0-100)

Berikut adalah panduan langkah demi langkah untuk menyiapkan aplikasi dari awal hingga siap digunakan.

### 4.1. Prasyarat

Sebelum memulai, pastikan Anda memiliki:

- Node.js (versi 16 atau lebih baru) dan npm terinstal.
- Akun Supabase (https://supabase.io).
- Akun OpenRouter (https://openrouter.ai) untuk API key.
- Akun Cloudflare untuk membuat Worker.
- Akun Vercel untuk hosting.
- Git dan akun GitHub untuk version control.

### 4.2. Installasi Project React.js

**Tujuan:** Membuat proyek React.js dan menginstal dependensi yang diperlukan.

1. Buka terminal dan jalankan perintah berikut untuk membuat proyek React baru:

   ```bash
   npx create-react-app ai-app
   cd ai-app
   ```

2. Instal dependensi tambahan:

   ```bash
   npm install @supabase/supabase-js axios
   ```

3. Selesai! Proyek React Anda sekarang siap untuk konfigurasi lebih lanjut.

### 4.3. Setup Supabase Project

**Tujuan:** Membuat proyek di Supabase untuk autentikasi dan database.

1. Buka Supabase Dashboard.
2. Klik **New Project** dan isi detail proyek (nama, organisasi, dll.).
3. Setelah proyek dibuat:
   - Navigasi ke **Settings &gt; API**.
   - Catat **Project URL** dan **Anon Public Key**.
4. Aktifkan autentikasi Email/Password:
   - Buka **Authentication &gt; Settings**.
   - Aktifkan opsi **Email/Password login**.
5. Supabase Anda sekarang siap untuk digunakan!

### 4.4. Buat Database Table (Chat & Message)

**Tujuan:** Membuat tabel database untuk menyimpan data chat dan pesan.

1. Di Supabase Dashboard, buka **SQL Editor**.
2. Jalankan skrip SQL berikut untuk membuat tabel:

```sql
-- Table Chat
CREATE TABLE chats (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users ON DELETE CASCADE,
  title TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Table Message
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  chat_id UUID REFERENCES chats(id) ON DELETE CASCADE,
  sender TEXT, -- 'user' atau 'ai'
  content TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

3. Pastikan tabel telah dibuat dengan memeriksa di **Table Editor**.
4. Database Anda sekarang siap untuk menyimpan data chat!

### 4.5. Setup Supabase Client di React

**Tujuan:** Menghubungkan aplikasi React dengan Supabase.

1. Buat file baru di `src/supabaseClient.js` dengan konten berikut:

```javascript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = 'https://your-project-id.supabase.co';
const supabaseAnonKey = 'your-anon-key';

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

2. Ganti `your-project-id` dan `your-anon-key` dengan nilai dari Supabase Dashboard (Settings &gt; API).
3. Supabase client Anda sekarang terhubung dengan frontend!

### 4.6. Setup Cloudflare Worker (Proxy API)

**Tujuan:** Membuat proxy API untuk mengamankan API key OpenRouter.

1. Buka Cloudflare Dashboard dan navigasi ke **Workers**.
2. Klik **Create Worker** dan beri nama Worker Anda.
3. Salin dan tempel kode berikut ke editor Worker:

```javascript
export default {
  async fetch(request) {
    const apiUrl = 'https://openrouter.ai/api/v1/chat/completions';
    const apiKey = 'Bearer YOUR_OPENROUTER_API_KEY';

    const newRequest = new Request(apiUrl, {
      method: request.method,
      headers: {
        'Authorization': apiKey,
        'Content-Type': 'application/json',
      },
      body: request.body,
    });

    return fetch(newRequest);
  }
}
```

4. Ganti `YOUR_OPENROUTER_API_KEY` dengan API key dari OpenRouter.
5. Klik **Deploy** dan catat URL Worker (contoh: `https://your-worker.pages.dev`).
6. Worker Anda sekarang siap untuk meneruskan permintaan ke OpenRouter!

### 4.7. Koding Sederhana: Login dan Chat

**Tujuan:** Membuat halaman login dan chat untuk aplikasi.

#### 4.7.1. Login Page (src/Login.js)

Buat file `src/Login.js` dengan kode berikut:

```javascript
import { supabase } from './supabaseClient';
import { useState } from 'react';

function Login({ onLogin }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  async function handleLogin() {
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    if (!error) onLogin();
  }

  async function handleSignUp() {
    const { error } = await supabase.auth.signUp({ email, password });
    if (!error) onLogin();
  }

  return (
    <div>
      <input placeholder="Email" onChange={(e) => setEmail(e.target.value)} />
      <input placeholder="Password" type="password" onChange={(e) => setPassword(e.target.value)} />
      <button onClick={handleLogin}>Login</button>
      <button onClick={handleSignUp}>Sign Up</button>
    </div>
  );
}

export default Login;
```

**Penjelasan:**

- Komponen ini menangani login dan pendaftaran menggunakan Supabase Auth.
- Input email dan password dikirim ke Supabase untuk autentikasi.
- Jika berhasil, fungsi `onLogin` dipanggil untuk mengarahkan ke halaman chat.

#### 4.7.2. Chat Page (src/Chat.js)

Buat file `src/Chat.js` dengan kode berikut:

```javascript
import { supabase } from './supabaseClient';
import { useState, useEffect } from 'react';
import axios from 'axios';

function Chat() {
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState([]);
  const [chatId, setChatId] = useState(null);

  useEffect(() => {
    createNewChat();
  }, []);

  async function createNewChat() {
    const { data } = await supabase.from('chats').insert([{ title: 'New Chat' }]).select().single();
    setChatId(data.id);
  }

  async function sendMessage() {
    if (!chatId) return;

    // Simpan pesan pengguna
    await supabase.from('messages').insert([{ chat_id: chatId, sender: 'user', content: input }]);

    // Kirim permintaan ke AI melalui Cloudflare Worker
    const response = await axios.post('https://your-worker.pages.dev', {
      model: 'gpt-4',
      messages: [{ role: 'user', content: input }],
    });

    const aiReply = response.data.choices[0].message.content;

    // Simpan pesan AI
    await supabase.from('messages').insert([{ chat_id: chatId, sender: 'ai', content: aiReply }]);

    // Perbarui UI
    setMessages([...messages, { sender: 'user', content: input }, { sender: 'ai', content: aiReply }]);
    setInput('');
  }

  return (
    <div>
      <div>
        {messages.map((msg, idx) => (
          <div key={idx}>
            <b>{msg.sender}:</b> {msg.content}
          </div>
        ))}
      </div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={sendMessage}>Send</button>
    </div>
  );
}

export default Chat;
```

**Penjelasan:**

- Komponen ini menangani pembuatan sesi chat baru dan pengiriman pesan.
- Pesan pengguna disimpan di database, dikirim ke OpenRouter melalui Cloudflare Worker, dan respons AI disimpan kembali.
- UI diperbarui secara real-time untuk menampilkan pesan.

#### 4.7.3. App Router (src/App.js)

Buat file `src/App.js` dengan kode berikut:

```javascript
import { useState } from 'react';
import Login from './Login';
import Chat from './Chat';

function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  return (
    <div>
      {isLoggedIn ? <Chat /> : <Login onLogin={() => setIsLoggedIn(true)} />}
    </div>
  );
}

export default App;
```

**Penjelasan:**

- Komponen ini mengatur routing sederhana: menampilkan halaman login jika pengguna belum login, atau halaman chat jika sudah login.
- State `isLoggedIn` digunakan untuk mengontrol tampilan.

### 4.8. Hosting ke Vercel

**Tujuan:** Mendeploy aplikasi ke Vercel untuk akses publik.

1. Push proyek ke GitHub:

   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/yourname/ai-app.git
   git push -u origin main
   ```

2. Buka Vercel Dashboard dan lakukan langkah berikut:

   - Klik **Import Project** dan pilih repositori GitHub Anda.
   - Tambahkan **Environment Variables** di pengaturan proyek:
     - `REACT_APP_SUPABASE_URL`: URL proyek Supabase Anda.
     - `REACT_APP_SUPABASE_ANON_KEY`: Anon Public Key dari Supabase.
     - `REACT_APP_WORKER_URL`: URL Cloudflare Worker Anda.
   - Klik **Deploy**.

3. Setelah deploy selesai, Vercel akan memberikan URL publik untuk aplikasi Anda (contoh: `https://ai-app.vercel.app`).

---

## 5. Struktur Project

Berikut adalah struktur direktori proyek setelah setup:

```
/ai-app
├── /src
│   ├── App.js
│   ├── Login.js
│   ├── Chat.js
│   ├── supabaseClient.js
├── README.md
├── package.json
```

**Penjelasan:**

- `src/` berisi semua kode sumber aplikasi.
- `App.js` adalah komponen utama untuk routing.
- `Login.js` dan `Chat.js` menangani antarmuka pengguna.
- `supabaseClient.js` mengatur koneksi ke Supabase.
- `README.md` berisi dokumentasi ini.
- `package.json` berisi dependensi dan skrip proyek.

---

## 6. Selesai!

Selamat! Anda telah berhasil membangun dan mendeploy **AI Web App** dengan fitur berikut:

- Sistem autentikasi pengguna yang aman menggunakan Supabase.
- Fitur chat interaktif dengan AI melalui OpenRouter.
- Penyimpanan pesan pengguna dan AI di database Supabase.
- Hosting cepat dan aman di Vercel.

Aplikasi ini siap untuk digunakan dalam lingkungan produksi dan dapat dikembangkan lebih lanjut sesuai kebutuhan.

---

## 7. Catatan Tambahan

Untuk menjaga aplikasi tetap aman dan optimal, perhatikan hal berikut:

- **Jangan pernah** menyimpan API key OpenRouter di frontend. Selalu gunakan Cloudflare Worker untuk mengamankan permintaan API.
- Pilih model AI di OpenRouter sesuai kebutuhan (contoh: `gpt-4`, `gpt-3.5`, `llama`, dll.).
- Lakukan backup database Supabase secara berkala untuk mencegah kehilangan data.
- Untuk pengembangan lebih lanjut, Anda dapat menambahkan fitur seperti:
  - Mengganti nama sesi chat.
  - Menghapus sesi chat.
  - Melihat histori chat.
  - Memilih model AI secara dinamis.

---

## 8. Pengembangan Lanjutan (Opsional)

Jika Anda ingin meningkatkan aplikasi ini, berikut adalah beberapa saran:

- **Versi Pro:** Tambahkan fitur multi-chat room, histori chat yang dapat diakses, dan pemilihan model AI secara langsung oleh pengguna.
- **Styling:** Gunakan Tailwind CSS atau library UI seperti Material-UI untuk membuat antarmuka lebih menarik.
- **Fitur Tambahan:** Implementasikan notifikasi real-time menggunakan Supabase Realtime atau tambahkan fitur ekspor chat ke PDF.

Jika Anda membutuhkan bantuan untuk mengimplementasikan fitur-fitur ini, silakan beri tahu!

---

## 9. Kontribusi

Jika Anda ingin berkontribusi pada proyek ini:

1. Fork repositori di GitHub.
2. Buat branch baru untuk fitur atau perbaikan (`git checkout -b feature/nama-fitur`).
3. Commit perubahan Anda (`git commit -m "Menambahkan fitur X"`).
4. Push ke branch Anda (`git push origin feature/nama-fitur`).
5. Buat Pull Request di GitHub.

---

## 10. Lisensi

Proyek ini dilisensikan di bawah MIT License. Anda bebas menggunakan, memodifikasi, dan mendistribusikan kode ini sesuai kebutuhan.

---

**Terima kasih telah menggunakan AI Web App!** Jika Anda memiliki pertanyaan atau membutuhkan bantuan lebih lanjut, jangan ragu untuk menghubungi.