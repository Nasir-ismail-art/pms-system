```markdown
# PMS Starter (Next.js + Tailwind + Supabase)

هذا المشروع هو Starter لإدارة ممتلكات (PMS) مبني بـ Next.js + Tailwind + Supabase.

المتطلبات
- Node.js v18+
- حساب Supabase

تشغيل محلي
1. انسخ المشروع أو أنشئ مجلد جديد.
2. `npm install`
3. أنشئ ملف `.env.local` وضع القيم التالية:
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
```
4. `npm run dev`

هيكل المشروع
- /pages
- /pages/api
- /components
- /lib
- /styles

ملاحظات
- قبل تشغيل endpoints تأكّد من تهيئة الجداول في Supabase (ملف supabase_schema.sql مرفق).
- هذا Starter بسيط: يحتوي على Dashboard وAPI للحجوزات، ويمكن توسيعه لخصائص CRUD، Auth، صفحات إدارة، وتقارير.
```
```


```name=package.json
{
  "name": "pms-nasir",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@supabase/supabase-js": "^2.0.0",
    "next": "13.4.10",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "swr": "^2.1.0"
  },
  "devDependencies": {
    "autoprefixer": "^10.4.14",
    "postcss": "^8.4.24",
    "tailwindcss": "^3.4.7"
  }
}
```


```name=next.config.js
/** Simple Next.js config */
const nextConfig = {
  reactStrictMode: true,
}

module.exports = nextConfig
```


```name=postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```


```name=tailwind.config.js
module.exports = {
  content: ["./pages/**/*.{js,ts,jsx,tsx}", "./components/**/*.{js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [],
}
```


```name=.gitignore
node_modules
.env.local
.next
```


```name=.env.example
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
```


```name=styles/globals.css
@tailwind base;
@tailwind components;
@tailwind utilities;

html, body, #__next { height: 100%; }
body { @apply bg-gray-50 text-gray-900; }
```


```name=lib/supabaseClient.js
import { createClient } from '@supabase/supabase-js'

const SUPABASE_URL = process.env.NEXT_PUBLIC_SUPABASE_URL
const SUPABASE_ANON_KEY = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY

export const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY)
```


```name=pages/_app.js
import '../styles/globals.css'

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />
}
```


```name=pages/index.js
import BookingsList from '../components/BookingsList'

export default function Home() {
  return (
    <main className="max-w-5xl mx-auto p-6">
      <h1 className="text-2xl font-bold mb-4">PMS Dashboard</h1>

      <section className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
        <div className="p-4 bg-white rounded shadow">إجمالي الدخل<br/><span className="text-xl font-semibold">—</span></div>
        <div className="p-4 bg-white rounded shadow">نسبة الإشغال<br/><span className="text-xl font-semibold">—</span></div>
        <div className="p-4 bg-white rounded shadow">شكاوى/صيانة مفتوحة<br/><span className="text-xl font-semibold">—</span></div>
      </section>

      <section className="bg-white p-4 rounded shadow">
        <BookingsList />
      </section>
    </main>
  )
}
```


```name=components/BookingsList.jsx
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then(r => r.json())

export default function BookingsList() {
  const { data, error } = useSWR('/api/bookings', fetcher)

  if (error) return <div className="p-4">خطأ في تحميل الحجوزات</div>
  if (!data) return <div className="p-4">جاري التحميل...</div>

  return (
    <div>
      <h2 className="text-lg font-semibold mb-2">الحجوزات الأخيرة</h2>
      <ul>
        {data.bookings.map(b => (
          <li key={b.id} className="p-3 border-b flex justify-between items-center">
            <div>
              <div className="font-medium">{b.property_id}</div>
              <div className="text-sm text-gray-600">{new Date(b.start_date).toLocaleDateString()} — {new Date(b.end_date).toLocaleDateString()}</div>
            </div>
            <div className="text-right">
              <div className="font-semibold">{b.total_amount} EGP</div>
              <div className="text-sm text-gray-600">{b.status}</div>
            </div>
          </li>
        ))}
      </ul>
    </div>
  )
}
```


```name=pages/api/bookings.js
import { supabase } from '../../lib/supabaseClient'

export default async function handler(req, res) {
  if (req.method === 'GET') {
    const { data, error } = await supabase
      .from('bookings')
      .select('*')
      .order('start_date', { ascending: false })

    if (error) return res.status(500).json({ error: error.message })
    return res.status(200).json({ bookings: data || [] })
  }

  if (req.method === 'POST') {
    const { property_id, guest_id, start_date, end_date, total_amount, platform } = req.body
    const { data, error } = await supabase
      .from('bookings')
      .insert([{ property_id, guest_id, start_date, end_date, total_amount, platform }])

    if (error) return res.status(500).json({ error: error.message })
    return res.status(200).json({ booking: data[0] })
  }

  res.status(405).json({ error: 'Method not allowed' })
}
```


```name=supabase_schema.sql
-- profiles
create table profiles (
  id uuid primary key references auth.users(id),
  full_name text,
  phone text,
  role text default 'staff',
  created_at timestamptz default now()
);

-- properties
create table properties (
  id uuid primary key default gen_random_uuid(),
  owner_id uuid references profiles(id),
  title text,
  address text,
  description text,
  nightly_price numeric,
  created_at timestamptz default now()
);

-- bookings
create table bookings (
  id uuid primary key default gen_random_uuid(),
  property_id uuid references properties(id),
  guest_id uuid references profiles(id),
  start_date date,
  end_date date,
  total_amount numeric,
  platform text,
  status text default 'confirmed',
  created_at timestamptz default now()
);

-- expenses
create table expenses (
  id uuid primary key default gen_random_uuid(),
  property_id uuid references properties(id),
  amount numeric,
  category text,
  note text,
  expense_date date default current_date,
  created_at timestamptz default now()
);

-- maintenance
create table maintenance (
  id uuid primary key default gen_random_uuid(),
  property_id uuid references properties(id),
  title text,
  description text,
  status text default 'open',
  assigned_to uuid references profiles(id),
  cost numeric,
  reported_at timestamptz default now(),
  resolved_at timestamptz
);

-- housekeeping
create table housekeeping (
  id uuid primary key default gen_random_uuid(),
  property_id uuid references properties(id),
  date date,
  cleaner_id uuid references profiles(id),
  status text,
  notes text,
  created_at timestamptz default now()
);

-- reviews
create table reviews (
  id uuid primary key default gen_random_uuid(),
  property_id uuid references properties(id),
  guest_id uuid references profiles(id),
  rating int check (rating >= 1 and rating <= 5),
  comment text,
  created_at timestamptz default now()
);
```
