# share
# KnemOS — Person 1 Build Prompt
# Role: Website (Next.js 15) + Authentication

> Paste this entire file into your AI coding assistant (Antigravity / Claude Code / Cursor).
> Read every section before generating code.

---

## YOUR ROLE IN THE TEAM

You are building the **public-facing website and authentication layer** for KnemOS.

Your deliverable is a deployed Next.js 15 website at `knemos.vercel.app` that:
1. Markets and explains KnemOS to visitors
2. Authenticates users via Supabase
3. Provides app download and extension install links
4. Uses the **minimalwhite-demo.vercel.app** template as its design base

**You DO NOT build the desktop app or backend.**
**You DO NOT build the Chrome extension.**

Your integration with teammates:
- **Person 2** defines the API. You do NOT call `localhost:8765` from the website (that's a local machine address — users won't have it). You only call Supabase from the website.
- **Person 3** builds the extension. You provide the extension install link on the `/download` page.
- You pass a Supabase JWT to the desktop app via a deep link: `knemos://auth?token=JWT_HERE`

---

## CRITICAL FIRST STEP

You already have the **minimalwhite-demo.vercel.app** template source code.
**DO NOT rebuild it from scratch.**

The workflow is:
1. Open the template source
2. Replace the design tokens (colors, fonts) with KnemOS values
3. Replace each section's content with KnemOS copy
4. Add new sections that don't exist in the template (Stats, Features, Tech, Download CTA)
5. Add the auth flow
6. Deploy to Vercel

---

## TECH STACK

```
Framework:      Next.js 15 (App Router)
Language:       TypeScript
Styling:        TailwindCSS v3 (already in template)
Animation:      Framer Motion (already in template)
Auth:           @supabase/supabase-js + @supabase/ssr
Fonts:          Space Grotesk (headers) + Inter (body)
Icons:          lucide-react
Deployment:     Vercel
```

Install only NEW dependencies (don't reinstall what template already has):
```bash
npm install @supabase/supabase-js @supabase/ssr lucide-react
npm install --save-dev @types/node
```

---

## DESIGN SYSTEM (Override Template Tokens)

Add these to `globals.css` or `tailwind.config.ts`:

```css
/* globals.css — KnemOS design tokens */
:root {
  --color-black:       #0A0A0A;
  --color-white:       #FFFFFF;
  --color-mint:        #00C896;
  --color-mint-dark:   #009B74;
  --color-gray:        #888888;
  --color-light-gray:  #F2F2F2;
  --color-dark-card:   #111111;
  --color-text-mid:    #444444;
  --color-text-light:  #BBBBBB;
  --color-border:      #E8E8E8;
}
```

```ts
// tailwind.config.ts — extend with KnemOS palette
extend: {
  colors: {
    mint: '#00C896',
    'mint-dark': '#009B74',
    'dark-card': '#111111',
    'text-mid': '#444444',
    'text-light': '#BBBBBB',
  },
  fontFamily: {
    display: ['Space Grotesk', 'Inter', 'sans-serif'],
    body: ['Inter', 'sans-serif'],
    mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
  }
}
```

**Key color changes from template:**
- Template's primary button (black): keep as-is
- Template accent: replace with `#00C896` (mint)
- Template's hero headline font: keep the same weight/size, change content only
- Background: pure white `#FFFFFF` on light sections, `#0A0A0A` on dark sections

---

## PROJECT STRUCTURE

```
website/
├── app/
│   ├── layout.tsx              # Root layout, fonts, metadata
│   ├── page.tsx                # Landing page (imports all sections)
│   ├── auth/
│   │   ├── page.tsx            # Login/Signup
│   │   └── callback/
│   │       └── route.ts        # Supabase OAuth callback
│   └── download/
│       └── page.tsx            # App + extension download page
│
├── components/
│   ├── sections/               # Landing page sections
│   │   ├── Hero.tsx
│   │   ├── Stats.tsx
│   │   ├── Solution.tsx
│   │   ├── Features.tsx
│   │   ├── HowItWorks.tsx
│   │   ├── DownloadCTA.tsx
│   │   └── Footer.tsx
│   │
│   ├── ui/
│   │   ├── Navbar.tsx
│   │   ├── MintButton.tsx      # Primary CTA button (mint bg)
│   │   ├── DarkButton.tsx      # Secondary CTA (black bg)
│   │   ├── FeatureCard.tsx     # Feature grid card
│   │   └── StatCounter.tsx     # Animated number counter
│   │
│   └── auth/
│       ├── AuthForm.tsx        # Login/signup form
│       └── AuthProvider.tsx    # Supabase session context
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts           # Browser Supabase client
│   │   └── server.ts           # Server Supabase client
│   └── constants.ts            # App-wide constants
│
├── public/
│   ├── logo.svg                # KnemOS hexagon logo (ask Person 2 for SVG)
│   ├── logo-full.png           # KnemOS logo with wordmark
│   └── preview.png             # App screenshot for OG image
│
└── .env.local
```

---

## SECTION-BY-SECTION IMPLEMENTATION

### Section 1: Navbar

Adapt the template navbar. Replace placeholder logo with KnemOS.

```tsx
// components/ui/Navbar.tsx
'use client'
import Link from 'next/link'
import { usePathname } from 'next/navigation'

export const Navbar = () => (
  <nav className="fixed top-0 w-full z-50 bg-white/90 backdrop-blur-sm border-b border-[#F0F0F0]">
    <div className="max-w-6xl mx-auto px-6 h-14 flex items-center justify-between">

      {/* Logo */}
      <Link href="/" className="flex items-center gap-2">
        <img src="/logo.svg" alt="KnemOS" className="w-7 h-7" />
        <span className="font-bold text-sm tracking-widest text-black">KnemOS</span>
      </Link>

      {/* Nav links */}
      <div className="hidden md:flex items-center gap-8 text-sm text-[#444444]">
        <Link href="#features" className="hover:text-black transition-colors">Features</Link>
        <Link href="#how-it-works" className="hover:text-black transition-colors">How It Works</Link>
        <Link href="/download" className="hover:text-black transition-colors">Download</Link>
        <Link href="/auth" className="bg-black text-white px-4 py-1.5 text-xs tracking-widest uppercase hover:bg-[#111] transition-colors">
          Sign In
        </Link>
      </div>
    </div>
  </nav>
)
```

---

### Section 2: Hero (Adapt Template Hero)

This maps DIRECTLY to the template's hero section. Keep all the floating shapes, animations, and layout. Only change the content.

```tsx
// components/sections/Hero.tsx
'use client'
import { motion } from 'framer-motion'
import Link from 'next/link'

export const Hero = () => (
  <section className="relative min-h-screen bg-white flex flex-col items-center justify-center overflow-hidden">

    {/* Floating geometric shapes — KEEP FROM TEMPLATE, just adjust colors to match */}
    {/* Template has: thin circle outline, diamond, small rectangle outline */}
    {/* Keep all these exactly as template, just ensure they're opacity-[0.12] */}

    <div className="relative z-10 text-center px-6 max-w-4xl mx-auto">

      {/* Badge */}
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6 }}
        className="inline-flex items-center gap-2 text-xs text-[#888] tracking-[0.25em] uppercase mb-8 border border-[#E8E8E8] px-4 py-2"
      >
        <span className="w-1.5 h-1.5 rounded-full bg-mint animate-pulse" />
        OSC AI Build 1.0 — Future of Productivity
      </motion.div>

      {/* Main Headline — same font size as template "Less is More" */}
      <motion.h1
        initial={{ opacity: 0, y: 30 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.7, delay: 0.1 }}
        className="text-6xl md:text-8xl font-bold text-black leading-[0.95] tracking-tight mb-6"
        style={{ fontFamily: 'Space Grotesk, sans-serif' }}
      >
        Less Context
        <br />
        Switching.
      </motion.h1>

      {/* Subheadline */}
      <motion.p
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6, delay: 0.2 }}
        className="text-sm tracking-[0.3em] uppercase text-[#888888] mb-6"
      >
        AI-Powered Semantic Workspace Operating System
      </motion.p>

      {/* Body */}
      <motion.p
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6, delay: 0.3 }}
        className="text-base text-[#555555] max-w-xl mx-auto leading-relaxed mb-10"
      >
        KnemOS automatically organizes your browser tabs, IDE sessions, and local files
        into intelligent semantic workspaces. Your screen history becomes searchable.
        Your focus becomes measurable.
      </motion.p>

      {/* CTA Buttons — use template's exact button style */}
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        transition={{ duration: 0.6, delay: 0.4 }}
        className="flex flex-col sm:flex-row gap-4 items-center justify-center"
      >
        {/* Primary — use template's dark button */}
        <Link
          href="/download"
          className="bg-black text-white px-8 py-3.5 text-sm tracking-widest uppercase hover:bg-[#111] transition-colors"
        >
          Download for Windows
        </Link>

        {/* Secondary */}
        <a
          href="https://github.com/your-org/KnemOS"
          className="text-sm text-[#888] hover:text-black transition-colors flex items-center gap-1"
        >
          View on GitHub <span>→</span>
        </a>
      </motion.div>
    </div>

    {/* Template's scroll indicator — keep exactly as-is */}
    {/* Thin vertical line with downward animation */}
  </section>
)
```

---

### Section 3: Stats (NEW — Add After Hero)

Dark section with animated count-up numbers. This is a new section not in the template.

```tsx
// components/sections/Stats.tsx
'use client'
import { useEffect, useRef, useState } from 'react'
import { useInView } from 'framer-motion'

const stats = [
  { value: 40,   suffix: '+',  label: 'Browser tabs open, average session' },
  { value: 20,   suffix: ' min', label: 'Lost daily to context switching' },
  { value: 4.3,  suffix: ' GB', label: 'RAM wasted on idle background tabs' },
  { value: 40,   suffix: '%',  label: 'Deep work efficiency destroyed' },
]

function CountUp({ target, suffix }: { target: number; suffix: string }) {
  const [count, setCount] = useState(0)
  const ref = useRef(null)
  const isInView = useInView(ref, { once: true })

  useEffect(() => {
    if (!isInView) return
    const duration = 1800
    const steps = 60
    const increment = target / steps
    let current = 0
    const timer = setInterval(() => {
      current = Math.min(current + increment, target)
      setCount(Math.round(current * 10) / 10)
      if (current >= target) clearInterval(timer)
    }, duration / steps)
    return () => clearInterval(timer)
  }, [isInView, target])

  return (
    <span ref={ref} className="font-mono">
      {count}{suffix}
    </span>
  )
}

export const Stats = () => (
  <section className="bg-[#0A0A0A] py-24 px-6">
    <div className="max-w-5xl mx-auto">

      <p className="text-xs tracking-[0.3em] uppercase text-mint mb-12 text-center">
        The Problem
      </p>

      <h2 className="text-4xl md:text-5xl font-bold text-white text-center mb-16 leading-tight">
        Tab Hell is Real.
      </h2>

      <div className="grid grid-cols-2 md:grid-cols-4 gap-8">
        {stats.map((s) => (
          <div key={s.label} className="text-center">
            <div className="text-5xl md:text-6xl font-bold text-white mb-3 font-mono">
              <CountUp target={s.value} suffix={s.suffix} />
            </div>
            <p className="text-sm text-[#888888] leading-snug">{s.label}</p>
          </div>
        ))}
      </div>
    </div>
  </section>
)
```

---

### Section 4: Solution

White section explaining what KnemOS does, with a simple animated diagram.

```tsx
// components/sections/Solution.tsx
'use client'
import { motion } from 'framer-motion'

export const Solution = () => (
  <section className="bg-white py-28 px-6">
    <div className="max-w-5xl mx-auto">

      <p className="text-xs tracking-[0.3em] uppercase text-[#888] mb-6 text-center">
        The Solution
      </p>

      <h2 className="text-4xl md:text-5xl font-bold text-black text-center mb-6 leading-tight max-w-2xl mx-auto">
        KnemOS understands<br />what you're working on.
      </h2>

      <p className="text-base text-[#555] text-center max-w-xl mx-auto mb-16 leading-relaxed">
        KnemOS reads every open window, browser tab, and file path.
        It groups them into named semantic workspaces automatically —
        no setup, no folders, no manual tagging.
      </p>

      {/* Transformation diagram */}
      <div className="flex items-center justify-center gap-6 flex-wrap">

        {/* Before */}
        <div className="space-y-1.5">
          {['github.com/VendorBridge', 'VS Code — auth.py', 'FastAPI Docs', 'YouTube (Tab #27)', 'Gmail — 4 tabs', 'Stack Overflow #3'].map((t) => (
            <div key={t} className="text-xs bg-[#F2F2F2] text-[#888] px-3 py-1.5 rounded opacity-70 font-mono">{t}</div>
          ))}
        </div>

        {/* Arrow + Logo */}
        <div className="flex flex-col items-center gap-2 px-4">
          <div className="w-10 h-10 bg-black rounded flex items-center justify-center">
            <img src="/logo.svg" alt="" className="w-6 h-6" />
          </div>
          <span className="text-2xl text-[#CCC]">→</span>
        </div>

        {/* After */}
        <div className="space-y-3">
          {[
            { name: '● VendorBridge Dev', items: ['GitHub · FastAPI · auth.py'] },
            { name: '○ Communication', items: ['Slack · Gmail'] },
            { name: '○ Noise', items: ['YouTube — hidden'] },
          ].map((ws) => (
            <div key={ws.name} className="bg-[#F8F8F8] border border-[#E8E8E8] px-4 py-2.5 rounded min-w-[200px]">
              <p className="text-xs font-bold text-black mb-1">{ws.name}</p>
              {ws.items.map(i => <p key={i} className="text-[10px] text-[#888]">{i}</p>)}
            </div>
          ))}
        </div>
      </div>
    </div>
  </section>
)
```

---

### Section 5: Features (6 Cards)

```tsx
// components/sections/Features.tsx
'use client'
import { motion } from 'framer-motion'

const features = [
  {
    num: '01',
    title: 'Semantic Clustering',
    desc: 'AI groups your tabs, files, and IDE windows into named workspaces automatically. Zero manual configuration.',
  },
  {
    num: '02',
    title: 'Memory Lane',
    desc: 'Natural language search over your entire screen history. "That auth bug from this morning" finds it instantly.',
  },
  {
    num: '03',
    title: 'Deep Work Mode',
    desc: 'AI detects off-context apps per workspace and minimizes them when you enter focus mode.',
  },
  {
    num: '04',
    title: 'RAM Recovery',
    desc: 'Inactive workspaces are hibernated. Live counter shows exactly how much memory has been reclaimed.',
  },
  {
    num: '05',
    title: 'Wolfram Analytics',
    desc: 'Cognitive Focus Score and workflow heatmaps powered by Wolfram Language. Understand how you actually work.',
  },
  {
    num: '06',
    title: 'Context Export',
    desc: 'One-click Markdown snapshot of any workspace — all links, file paths, and project context included.',
  },
]

export const Features = () => (
  <section id="features" className="bg-[#F8F8F8] py-28 px-6">
    <div className="max-w-5xl mx-auto">

      <p className="text-xs tracking-[0.3em] uppercase text-[#888] mb-6 text-center">Features</p>
      <h2 className="text-4xl font-bold text-black text-center mb-16">
        Everything your OS<br />should have been.
      </h2>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-5">
        {features.map((f, i) => (
          <motion.div
            key={f.num}
            initial={{ opacity: 0, y: 24 }}
            whileInView={{ opacity: 1, y: 0 }}
            viewport={{ once: true }}
            transition={{ delay: i * 0.08, duration: 0.5 }}
            whileHover={{ y: -4, boxShadow: '0 8px 24px rgba(0,0,0,0.08)' }}
            className="bg-white p-6 rounded-sm border border-[#E8E8E8] transition-shadow"
          >
            <span className="text-xs font-mono font-bold text-mint tracking-widest mb-4 block">
              {f.num}
            </span>
            <h3 className="text-base font-bold text-black mb-2">{f.title}</h3>
            <p className="text-sm text-[#666] leading-relaxed">{f.desc}</p>
          </motion.div>
        ))}
      </div>
    </div>
  </section>
)
```

---

### Section 6: How It Works (Tech Stack)

```tsx
// components/sections/HowItWorks.tsx
'use client'

const pipeline = [
  { step: '01', name: 'Data Collection', detail: 'pywin32 · watchdog · Chrome Extension' },
  { step: '02', name: 'Embeddings', detail: 'sentence-transformers / all-MiniLM-L6-v2' },
  { step: '03', name: 'Clustering', detail: 'HDBSCAN — density-based AI grouping' },
  { step: '04', name: 'Naming', detail: 'Ollama + Mistral 7B — local LLM' },
  { step: '05', name: 'Analytics', detail: 'Wolfram Language + wolframclient' },
]

export const HowItWorks = () => (
  <section id="how-it-works" className="bg-[#0A0A0A] py-28 px-6">
    <div className="max-w-5xl mx-auto">

      <p className="text-xs tracking-[0.3em] uppercase text-mint mb-6 text-center">Architecture</p>
      <h2 className="text-4xl font-bold text-white text-center mb-4">
        All intelligence. Zero cloud.
      </h2>
      <p className="text-sm text-[#888] text-center mb-16">
        All AI processing runs on{' '}
        <code className="font-mono text-mint">localhost:8765</code>.
        No data ever leaves your machine.
      </p>

      {/* Pipeline steps */}
      <div className="flex flex-col md:flex-row items-start md:items-center justify-between gap-4 mb-16">
        {pipeline.map((p, i) => (
          <div key={p.step} className="flex md:flex-col items-center gap-3 flex-1">
            <div className="flex-shrink-0 w-9 h-9 bg-[#1A1A1A] border border-[#2A2A2A] flex items-center justify-center">
              <span className="text-xs font-mono text-[#666]">{p.step}</span>
            </div>
            {i < pipeline.length - 1 && (
              <div className="flex-1 md:flex-none md:w-full h-px md:h-px bg-[#222] hidden md:block" />
            )}
            <div className="text-left md:text-center">
              <p className="text-xs font-bold text-white">{p.name}</p>
              <p className="text-[10px] text-[#666] mt-0.5">{p.detail}</p>
            </div>
          </div>
        ))}
      </div>

      {/* Tech stack grid */}
      <div className="grid grid-cols-2 md:grid-cols-4 gap-3">
        {['Tauri v2', 'React 18', 'FastAPI', 'ChromaDB', 'Ollama', 'Wolfram', 'HDBSCAN', 'Tesseract'].map((tech) => (
          <div key={tech} className="bg-[#111] border border-[#222] px-4 py-3 text-center">
            <span className="text-xs font-mono text-[#888]">{tech}</span>
          </div>
        ))}
      </div>
    </div>
  </section>
)
```

---

### Section 7: Download CTA

```tsx
// components/sections/DownloadCTA.tsx
import Link from 'next/link'

export const DownloadCTA = () => (
  <section className="bg-black py-28 px-6 text-center">
    <p className="text-xs tracking-[0.3em] uppercase text-mint mb-6">Get Started</p>
    <h2 className="text-4xl md:text-5xl font-bold text-white mb-6 leading-tight">
      Ready to end tab hell?
    </h2>
    <p className="text-sm text-[#666] mb-10">
      Free forever · Local-first · 100% Open Source
    </p>
    <div className="flex flex-col sm:flex-row gap-4 justify-center">
      <Link
        href="/download"
        className="bg-mint text-black px-8 py-4 text-sm font-bold tracking-widest uppercase hover:bg-mint-dark transition-colors"
      >
        Download for Windows
      </Link>
      <a
        href="https://github.com/your-org/KnemOS"
        className="border border-[#333] text-white px-8 py-4 text-sm tracking-widest uppercase hover:border-[#666] transition-colors"
      >
        View GitHub →
      </a>
    </div>
  </section>
)
```

---

### Section 8: Footer

```tsx
// components/sections/Footer.tsx
export const Footer = () => (
  <footer className="bg-white border-t border-[#F0F0F0] py-8 px-6">
    <div className="max-w-5xl mx-auto flex flex-col md:flex-row items-center justify-between gap-4">
      <div className="flex items-center gap-2">
        <img src="/logo.svg" alt="KnemOS" className="w-5 h-5" />
        <span className="text-sm font-bold tracking-widest">KnemOS</span>
      </div>
      <p className="text-xs text-[#888]">
        OSC AI Build 1.0 · Future of Productivity · MIT License
      </p>
      <div className="flex gap-6 text-xs text-[#888]">
        <a href="/download" className="hover:text-black">Download</a>
        <a href="https://github.com/your-org/KnemOS" className="hover:text-black">GitHub</a>
        <a href="/auth" className="hover:text-black">Sign In</a>
      </div>
    </div>
  </footer>
)
```

---

### Landing Page Assembly

```tsx
// app/page.tsx
import { Hero } from '@/components/sections/Hero'
import { Stats } from '@/components/sections/Stats'
import { Solution } from '@/components/sections/Solution'
import { Features } from '@/components/sections/Features'
import { HowItWorks } from '@/components/sections/HowItWorks'
import { DownloadCTA } from '@/components/sections/DownloadCTA'
import { Footer } from '@/components/sections/Footer'

export default function Home() {
  return (
    <main>
      <Hero />
      <Stats />
      <Solution />
      <Features />
      <HowItWorks />
      <DownloadCTA />
      <Footer />
    </main>
  )
}
```

---

## AUTHENTICATION IMPLEMENTATION

### Supabase Setup

```bash
# Install
npm install @supabase/supabase-js @supabase/ssr
```

```ts
// lib/supabase/client.ts — browser client
import { createBrowserClient } from '@supabase/ssr'

export const createClient = () =>
  createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
```

```ts
// lib/supabase/server.ts — server client (for Route Handlers)
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export const createClient = () => {
  const cookieStore = cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    { cookies: { getAll: () => cookieStore.getAll(), setAll: (cs) => cs.forEach(({ name, value, options }) => cookieStore.set(name, value, options)) } }
  )
}
```

### Auth Page

```tsx
// app/auth/page.tsx
'use client'
import { useState } from 'react'
import { createClient } from '@/lib/supabase/client'
import { useRouter } from 'next/navigation'

export default function AuthPage() {
  const [email, setEmail] = useState('')
  const [loading, setLoading] = useState(false)
  const [sent, setSent] = useState(false)
  const supabase = createClient()
  const router = useRouter()

  const handleMagicLink = async (e: React.FormEvent) => {
    e.preventDefault()
    setLoading(true)
    const { error } = await supabase.auth.signInWithOtp({
      email,
      options: {
        emailRedirectTo: `${window.location.origin}/auth/callback`,
      },
    })
    setLoading(false)
    if (!error) setSent(true)
  }

  const handleGoogle = async () => {
    await supabase.auth.signInWithOAuth({
      provider: 'google',
      options: { redirectTo: `${window.location.origin}/auth/callback` }
    })
  }

  return (
    <main className="min-h-screen bg-white flex items-center justify-center px-6">
      <div className="w-full max-w-sm">

        <div className="flex items-center gap-2 justify-center mb-10">
          <img src="/logo.svg" alt="KnemOS" className="w-8 h-8" />
          <span className="font-bold text-lg tracking-widest">KnemOS</span>
        </div>

        {sent ? (
          <div className="text-center">
            <p className="text-lg font-bold mb-2">Check your email</p>
            <p className="text-sm text-[#888]">We sent a magic link to <strong>{email}</strong></p>
          </div>
        ) : (
          <>
            <h1 className="text-2xl font-bold text-center mb-8">Sign In</h1>

            <form onSubmit={handleMagicLink} className="space-y-4">
              <input
                type="email"
                placeholder="your@email.com"
                value={email}
                onChange={e => setEmail(e.target.value)}
                required
                className="w-full border border-[#E0E0E0] px-4 py-3 text-sm outline-none focus:border-black transition-colors"
              />
              <button
                type="submit"
                disabled={loading}
                className="w-full bg-black text-white py-3 text-sm tracking-widest uppercase disabled:opacity-50"
              >
                {loading ? 'Sending...' : 'Continue with Email'}
              </button>
            </form>

            <div className="my-6 flex items-center gap-3">
              <div className="flex-1 h-px bg-[#E8E8E8]" />
              <span className="text-xs text-[#888]">or</span>
              <div className="flex-1 h-px bg-[#E8E8E8]" />
            </div>

            <button
              onClick={handleGoogle}
              className="w-full border border-[#E0E0E0] py-3 text-sm text-[#444] hover:border-black transition-colors"
            >
              Continue with Google
            </button>
          </>
        )}
      </div>
    </main>
  )
}
```

### Auth Callback (After Email/OAuth Redirect)

```ts
// app/auth/callback/route.ts
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const { searchParams, origin } = new URL(request.url)
  const code = searchParams.get('code')

  if (code) {
    const supabase = createClient()
    const { data, error } = await supabase.auth.exchangeCodeForSession(code)

    if (!error && data.session) {
      // After successful auth, redirect to download page
      // The download page will show the deep link to open the desktop app
      return NextResponse.redirect(`${origin}/download?token=${data.session.access_token}`)
    }
  }

  return NextResponse.redirect(`${origin}/auth?error=callback_failed`)
}
```

### Download Page with Deep Link

```tsx
// app/download/page.tsx
'use client'
import { useSearchParams } from 'next/navigation'
import { useEffect } from 'react'

export default function DownloadPage() {
  const params = useSearchParams()
  const token = params.get('token')

  useEffect(() => {
    // Auto-trigger deep link to open desktop app with auth token
    // Desktop app must register 'knemos://' protocol handler
    if (token) {
      setTimeout(() => {
        window.location.href = `knemos://auth?token=${token}`
      }, 1500)
    }
  }, [token])

  return (
    <main className="min-h-screen bg-white flex items-center justify-center px-6">
      <div className="max-w-lg text-center">

        <img src="/logo.svg" alt="KnemOS" className="w-16 h-16 mx-auto mb-8" />
        <h1 className="text-3xl font-bold mb-4">Download KnemOS</h1>

        {/* Desktop App Download */}
        <div className="border border-[#E0E0E0] p-6 mb-4 text-left">
          <h2 className="font-bold text-sm tracking-widest uppercase mb-1">Desktop App</h2>
          <p className="text-xs text-[#888] mb-4">Windows 10/11 — Requires Ollama + Tesseract</p>
          <a
            href="/downloads/KnemOS-Setup.exe"
            className="block w-full bg-black text-white text-center py-3 text-sm tracking-widest uppercase"
          >
            Download .exe (Windows)
          </a>
        </div>

        {/* Extension Install */}
        <div className="border border-[#E0E0E0] p-6 text-left">
          <h2 className="font-bold text-sm tracking-widest uppercase mb-1">Browser Extension</h2>
          <p className="text-xs text-[#888] mb-4">Chrome / Edge — Optional but recommended</p>
          <a
            href="https://chrome.google.com/webstore/detail/knemos/your-extension-id"
            className="block w-full bg-white border border-black text-black text-center py-3 text-sm tracking-widest uppercase hover:bg-black hover:text-white transition-colors"
          >
            Install Extension
          </a>
        </div>

        <p className="text-xs text-[#888] mt-6">
          Free forever · Local-first · MIT License
        </p>
      </div>
    </main>
  )
}
```

---

## ENVIRONMENT VARIABLES

```env
# .env.local
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
```

---

## INTEGRATION CONTRACT WITH PERSON 2 & 3

### What You Give to Person 2
```
1. Supabase Project URL + anon key (share in team chat, not in code)
2. Deep link format: knemos://auth?token=SUPABASE_JWT
   Person 2 must register this protocol in Tauri (src-tauri/tauri.conf.json)
3. Download page URL: https://knemos.vercel.app/download
```

### What You Give to Person 3
```
1. Extension Chrome Web Store link (placeholder for now)
   URL: https://chrome.google.com/webstore/detail/knemos/EXTENSION_ID
2. Their extension ID goes into your download page once it's published
```

### What Person 2 Gives You
```
1. Tauri deep link handler confirmation
   (They must add this to tauri.conf.json):
   "protocols": [{ "name": "knemos", "schemes": ["knemos"] }]

2. Confirmation that backend starts at http://127.0.0.1:8765
   (Website does NOT call this directly — this is just for your knowledge)
```

---

## DEPLOYMENT

```bash
# 1. Push to GitHub
git init && git add . && git commit -m "feat: KnemOS website"

# 2. Import to Vercel
# → vercel.com → New Project → Import GitHub repo

# 3. Set environment variables in Vercel dashboard:
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY

# 4. Deploy
# Vercel auto-deploys on every push to main
```

---

## FINAL CHECKLIST

```
□ Template adapted — not rebuilt from scratch
□ KnemOS brand tokens applied (mint color, fonts, logo)
□ All 7 sections complete (Hero, Stats, Solution, Features, HowItWorks, CTA, Footer)
□ Floating shapes visible in hero (from template)
□ Count-up animation on stats section works
□ Feature cards animate on scroll with stagger
□ Auth page: magic link sends email
□ Auth page: Google OAuth redirects correctly
□ /auth/callback route exchanges code for session
□ /download page shows both download options
□ Deep link knemos://auth?token= fires on /download page
□ Website deployed to Vercel
□ URL works on mobile (responsive design)
□ All links are not broken (GitHub placeholder is fine)
□ No console errors on production build
```

---

*KnemOS — OSC AI Build 1.0 | Person 1: Web UI + Auth*
