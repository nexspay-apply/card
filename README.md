# Cardwise: Netlify + Supabase

```
landing/
├─ netlify.toml            Netlify config (publish = public)
├─ supabase/
│  ├─ schema.sql           1) enquiries table + website insert-only security
│  └─ admin.sql            2) admin access, notes, realtime
└─ public/                 Ye folder live website hai
   ├─ index.html           Landing page
   ├─ apply.html           Application form  → Supabase insert
   ├─ admin.html           Admin panel (login, live data, Excel/PDF export)
   ├─ _headers             Security headers (Netlify)
   ├─ robots.txt           Admin page ko Google se chhupata hai
   └─ js/  config.js · apply.js · admin.js · main.js
```

## Step 1: Supabase
1. https://supabase.com par project banayein (region: Mumbai).
2. **SQL Editor** mein pehle `supabase/schema.sql` run karein, phir `supabase/admin.sql`.
3. **Authentication → Users → Add user → Create new user**
   - Email: `admin@cardwise.local` (ya apni email; `config.js` mein `ADMIN_EMAIL` wahi rakhein)
   - Password: strong password (8+ characters), aur **Auto Confirm User** ✔ karein
4. SQL Editor mein ye run karein, taaki ye user admin ban jaye:
   ```sql
   insert into public.admins (user_id)
   select id from auth.users where email = 'admin@cardwise.local'
   on conflict do nothing;
   ```
5. **Authentication → Sign In / Providers** mein "Allow new users to sign up" ko **OFF** kar dein.
6. **Project Settings → API** se Project URL aur `anon` key copy karke `public/js/config.js` mein daalein.

## Step 2: Netlify deploy
**Option A: drag and drop (sabse aasaan)**
https://app.netlify.com/drop kholein aur **`public` folder** drag karein.

**Option B: CLI** (PowerShell, `landing` folder ke andar)
```powershell
npm install -g netlify-cli
netlify login
netlify deploy --prod --dir=public
```

**Option C: GitHub** se repo connect karein. `netlify.toml` apne-aap `public` folder publish karega.

## Step 3: Use
- Website: `https://your-site.netlify.app`
- Form: `https://your-site.netlify.app/apply.html`
- Admin: `https://your-site.netlify.app/admin.html`
  User ID **admin** + aapka Supabase password

Admin panel features:
- Naya enquiry aate hi table mein turant dikhta hai (realtime).
- Search aur filters: status, interest, date range.
- Har row mein checkbox, aur "select all" sabhi filtered rows ek saath chun leta hai.
- **Excel** aur **PDF** export: rows select ki hon to sirf selected export hoti hain, warna sabhi filtered rows.
- Status badalna (New → Contacted → Applied → Closed), notes likhna, aur delete.

## Security
- Website (anon key) sirf naya data **daal** sakti hai; padh nahi sakti.
- Data sirf wahi dekh sakta hai jo `admins` table mein hai aur login kiya ho. Ye check database (RLS) karta hai, browser nahi.
- Password Supabase mein encrypted rehta hai, website ke code mein nahi.
