# **5. Arsitektur dan Implementasi Aplikasi Next.js**

## **5.1 Teknologi yang Digunakan**  

Aplikasi ini menggunakan serangkaian pustaka open-source berikut:  

### **Frontend**  

- **Next.js**: Framework React untuk pengembangan aplikasi web modern dengan dukungan *server-side rendering* dan *static site generation*.  
- **Tailwind CSS**: Pustaka utilitas CSS untuk membangun antarmuka secara cepat dan konsisten.  
- **ShadCN UI**: Komponen antarmuka berbasis Tailwind yang siap pakai dan dapat disesuaikan.  
- **React**: Pustaka JavaScript untuk membangun antarmuka pengguna berbasis komponen.  

### **Backend**  

- **Next.js**: Menyediakan API Routes dan Server Components untuk pengelolaan data dan backend.  
- **PostgreSQL**: Basis data relasional yang andal dan fleksibel untuk menyimpan data aplikasi.  
- **Prisma ORM**: Alat untuk mengelola akses database secara efisien menggunakan model berbasis TypeScript.  
- **Redis**: Penyimpanan data *in-memory* untuk caching dan pengelolaan sesi.  

### **Utilities**  

- **React Hook Form**: Pustaka untuk mengelola formulir dengan performa tinggi.  
- **Zod**: Pustaka validasi skema untuk memastikan data sesuai dengan spesifikasi.  
- **Zustand**: Pustaka manajemen state ringan untuk React.  

### **PDF Handling**  

- **PDFLib**: Pustaka untuk membuat dan memodifikasi PDF secara programatik.  
- **PDFKit**: Alat untuk menghasilkan file PDF dengan fitur lengkap, seperti grafik dan teks.  

### **Charting**  

- **Recharts**: Pustaka untuk membuat grafik interaktif berbasis React.  

### **Authentication**  

- **Auth.js**: Solusi autentikasi yang mendukung berbagai penyedia identitas dan strategi login.  

## **5.2 Pedoman Struktur dan Penulisan Kode**  

### **5.2.1 Penulisan Kode**

Kode ditulis secara modular untuk meningkatkan *readability* dan mempermudah pengelolaan. Setiap fungsi, komponen, atau modul dipisahkan dengan jelas sesuai dengan tanggung jawabnya sehingga mudah dipahami dan dimodifikasi.  

### **5.2.2 Struktur Folder**

1. **Komponen Lokal** yang hanya digunakan di bagian tertentu akan disimpan di folder `_components` di dalam direktori terdekat tempat komponen tersebut digunakan.  
2. **Kode Bersama** yang digunakan di berbagai bagian aplikasi ditempatkan di:
     - **`/src/lib`**: Untuk kode spesifik yang hanya relevan dengan aplikasi ini.  
     - **`/src/utils`**: Untuk kode yang bersifat umum atau dapat digunakan kembali di proyek lain.
3. **Skema Prisma** disimpan di folder `/src/prisma/db-honorarium/schema.prisma`
4. **Skema Zod** disimpan di folder `/src/zod/schemas`
5. **Custom hooks** disimpan di folder `/src/hooks`

### **5.2.3 Prinsip Akses Data**

- Untuk *client components*, data diambil dari server action yang berada di folder `/actions`.  
- Untuk *server components*, data diakses melalui fungsi-fungsi di folder `/data`.  
- Di folder `/actions`, penggunaan Prisma ORM untuk akses langsung ke database diminimalkan.  
- Jika memungkinkan, server action di `/actions` akan memanggil fungsi dari folder `/data`.  
- Semua fungsi di folder `/data` bertanggung jawab penuh atas query database menggunakan Prisma ORM, memastikan pengelolaan query terkonsentrasi dan terstruktur dengan baik.  

Pendekatan ini dirancang untuk menjaga konsistensi, skalabilitas, dan kemudahan dalam pengembangan serta pemeliharaan aplikasi.

## **5.3 Penjabaran Alur dan Fungsi Komponen Aplikasi**

### **5.3.1 Halaman Login**

#### **5.3.1.1 Route**

<https://d01.pirsani.id> atau <https://d01.pirsani.id/login>

#### **5.3.1.2 Tampilan visual**

![login](images/5/5-001-login.png)

gambar 5.1 halaman login

#### **5.3.1.2 Membedah login**

![login](images/5/disect-login.png)

gambar 5.2 halaman login dan file terkait

1. **page** `src\app\page.tsx` atau `src\app\(auth)\login\page.tsx`
2. **components:**
    - form `src\components\form\login\index.tsx`
    - input `src\components\form\login\input.tsx`
3. **login handler:**
    - `src\auth.ts`
    - `src\auth.config.ts`
  
pada saat pengguna melakukan login dan berhasil, sistem akan menyimpan data user satker dan roles di session.

```ts
# src\auth.config.ts

import { getNearestSatkerAnggaran } from "./data/organisasi";
import { getUser, getUserRoles } from "./data/user";


# kode lainnya ...

satkerAnggaran = await getNearestSatkerAnggaran(user.organisasiId);

# kode lainnya ...

const roles = await getUserRoles(user.id);
```

Session ini diextend dari default session library `Auth.js`.

```ts
# src\next-auth.d.ts

declare module "next-auth" {
  /**
   * Returned by `useSession`, `getSession` and received as a prop on the `SessionProvider` React Context
   */
  interface User {
    id: string;
    nip?: string | null;
    unitKerjaId?: string | null; // ini basically adalah organisasiId
    unitKerjaNama?: string | null;
    unitKerjaNamaSingkat?: string | null;
    satkerId?: string | null; // sedangkan ini adalah organisasiId terdekat yang merupakan satker anggaran
    satkerNama?: string | null;
    satkerNamaSingkat?: string | null;
    organisasiId?: string | null;
    organisasiNama?: string | null;
    organisasiNamaSingkat?: string | null;
    roles?: string[];
    permissions?: string[];
  }
  interface Session {
    user: User & DefaultSession["user"];
  }
}
```

penjelasan tentang cara extend session dapad dibaca di [https://authjs.dev/guides/extending-the-session](https://authjs.dev/guides/extending-the-session)

### **5.3.2 Middleware untuk pengecekan otentikasi**

referensi: <https://nextjs.org/docs/app/building-your-application/routing/middleware>

Middleware dijalankan sebelum sebuah request selesai. Pada saat pengguna mengakses aplikasi middleware akan melakukan pengecekan apakah user tersebut mempunyai sesi akses. jika belum maka akan diarahkan ke halaman login.


#### **5.3.2.1 file-file terkait middleware**

- `src\middleware.ts` : file utama yang akan dijalankan saat pengguna mengakses aplikasi
- `src\route.ts` : file ini mendefinisikan route untuk `dashboardRoutes`,`alurProsesRoutes`,`dataReferensiRoutes`

Agar middleware ini dapat dijalankan dengan ringan, maka middleware hanya melakukan pengecekan session(otentikasi). Proses untuk otorisasi akan dijalankan pada masing-masing halaman atau `server action`

```ts
# src\middleware.ts
    const { nextUrl } = req;
    const session = req.auth;
    const isLoggedIn = !!session;
```

### **5.3.2 Otorisasi**
