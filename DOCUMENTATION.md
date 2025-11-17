# Journey through Cinema - Technical Specification

## Project Overview

**Purpose**: Personal website exploring cinematic movements, directors, and films through essays and reviews  
**Stack**: Astro + React (islands) + TypeScript + Tailwind CSS  
**Languages**: Slovak (SK) and English (EN)  
**Deployment**: Vercel  
**Content Management**: MDX files (author-controlled)

---

## Architecture Overview

### Technology Stack

**Frontend Framework**:
- **Astro 4.x** - Static site generator, zero-JS by default
- **React 18+** - For interactive islands (search, dark mode, reading progress)
- **TypeScript** - Strict mode for type safety
- **Tailwind CSS** - Minimalist editorial styling

**Content**:
- **MDX** - Markdown with JSX for rich content
- **Content Collections API** - Type-safe content management
- **No CMS** - Direct file editing for full control

**Search & Discovery** (Phase 2):
- **Pagefind** - Static search, zero-config
- **Fuse.js** - Alternative client-side search

**Testing**:
- **Vitest** - Unit tests
- **@testing-library/react** - Component testing
- **Playwright** - E2E tests

**Development Tools**:
- **ESLint** - Astro + TypeScript linting
- **Prettier** - Code formatting
- **GitHub Actions** - CI/CD

---

## Project Structure

```
journey-cinema/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── public/
│   ├── images/
│   │   ├── films/
│   │   ├── directors/
│   │   ├── movements/
│   │   └── placeholder.jpg
│   ├── og-images/                # Generated Open Graph images
│   └── robots.txt
├── src/
│   ├── content/
│   │   ├── config.ts
│   │   └── articles/
│   │       ├── movements/        # Large articles
│   │       │   ├── italian-neorealism.mdx
│   │       │   └── french-new-wave.mdx
│   │       ├── directors/        # Medium articles
│   │       │   ├── robert-bresson.mdx
│   │       │   └── andrei-tarkovsky.mdx
│   │       └── films/            # Short reviews
│   │           ├── werckmeister-harmonies.mdx
│   │           └── tokyo-story.mdx
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Header.astro
│   │   │   ├── Footer.astro
│   │   │   └── LanguageSwitcher.astro
│   │   ├── articles/
│   │   │   ├── ArticleCard.astro
│   │   │   ├── ArticleList.astro
│   │   │   ├── TypeFilter.astro
│   │   │   └── RelatedArticles.astro  # Phase 2
│   │   ├── youtube/
│   │   │   ├── YouTubeCard.astro
│   │   │   └── YouTubeEmbed.astro
│   │   ├── ui/
│   │   │   ├── Button.astro
│   │   │   ├── DarkModeToggle.tsx     # React island
│   │   │   ├── ReadingProgress.tsx    # Phase 2
│   │   │   └── SearchBar.tsx          # Phase 3
│   │   └── forms/
│   │       └── ContactForm.tsx
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   ├── ArticleLayout.astro
│   │   └── PageLayout.astro
│   ├── pages/
│   │   ├── index.astro                # Custom introduction
│   │   ├── about.astro
│   │   ├── contact.astro
│   │   ├── sk/
│   │   │   ├── index.astro
│   │   │   ├── o-mne.astro
│   │   │   ├── kontakt.astro
│   │   │   ├── clanky/
│   │   │   │   ├── index.astro
│   │   │   │   └── [slug].astro
│   │   │   └── youtube-eseje.astro
│   │   ├── en/
│   │   │   ├── index.astro
│   │   │   ├── about.astro
│   │   │   ├── contact.astro
│   │   │   ├── articles/
│   │   │   │   ├── index.astro
│   │   │   │   └── [slug].astro
│   │   │   └── youtube-essays.astro
│   │   ├── tags/                      # Phase 2
│   │   │   ├── index.astro
│   │   │   └── [tag].astro
│   │   ├── search/                    # Phase 3
│   │   │   └── index.astro
│   │   └── rss.xml.ts                 # Phase 2
│   ├── api/
│   │   └── contact.ts
│   ├── data/
│   │   └── youtube-essays.ts          # Curated YouTube list
│   ├── lib/
│   │   ├── i18n.ts
│   │   ├── reading-time.ts
│   │   └── related-content.ts         # Phase 2
│   ├── styles/
│   │   └── global.css
│   ├── types/
│   │   ├── content.ts
│   │   └── youtube.ts
│   └── env.d.ts
├── tests/
│   ├── unit/
│   │   └── reading-time.test.ts
│   ├── integration/
│   │   └── ArticleCard.test.tsx
│   └── e2e/
│       ├── navigation.spec.ts
│       └── dark-mode.spec.ts
├── .env.example
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
├── vitest.config.ts
├── playwright.config.ts
├── package.json
└── README.md
```

---

## Phase 1: Core Content System

### 1.1 Content Schema Definition

**File**: `src/content/config.ts`

```typescript
import { defineCollection, z } from 'astro:content';

const articles = defineCollection({
  type: 'content',
  schema: ({ image }) =>
    z.object({
      title: z.string(),
      description: z.string(),
      publishDate: z.date(),
      type: z.enum(['movement', 'director', 'film']),
      language: z.enum(['sk', 'en']),
      coverImage: image().optional(),
      tags: z.array(z.string()).optional(),
      // Film-specific metadata
      director: z.string().optional(),
      year: z.number().optional(),
      country: z.string().optional(),
      runtime: z.string().optional(),
      // Draft mode
      draft: z.boolean().default(false),
    }),
});

export const collections = {
  articles,
};

// Type exports
export type Article = z.infer<typeof articles.schema>;
```

### 1.2 Internationalization Setup

**File**: `src/lib/i18n.ts`

```typescript
export const languages = {
  sk: 'Slovenčina',
  en: 'English',
} as const;

export type Language = keyof typeof languages;

export const defaultLang: Language = 'en';

export const ui = {
  sk: {
    'nav.home': 'Domov',
    'nav.about': 'O mne',
    'nav.articles': 'Články',
    'nav.contact': 'Kontakt',
    'nav.youtube': 'YouTube Eseje',
    'article.type.movement': 'Hnutie',
    'article.type.director': 'Režisér',
    'article.type.film': 'Film',
    'article.readTime': 'min čítania',
    'article.published': 'Publikované',
    'article.director': 'Réžia',
    'article.year': 'Rok',
    'article.country': 'Krajina',
    'article.runtime': 'Dĺžka',
    'filter.all': 'Všetky',
    'filter.movements': 'Hnutia',
    'filter.directors': 'Režiséri',
    'filter.films': 'Filmy',
    'search.placeholder': 'Hľadať články...',
    'tags.title': 'Tagy',
    'related.title': 'Súvisiace články',
    'contact.title': 'Kontakt',
    'contact.email': 'Email',
    'contact.message': 'Správa',
    'contact.submit': 'Odoslať',
  },
  en: {
    'nav.home': 'Home',
    'nav.about': 'About',
    'nav.articles': 'Articles',
    'nav.contact': 'Contact',
    'nav.youtube': 'YouTube Essays',
    'article.type.movement': 'Movement',
    'article.type.director': 'Director',
    'article.type.film': 'Film',
    'article.readTime': 'min read',
    'article.published': 'Published',
    'article.director': 'Directed by',
    'article.year': 'Year',
    'article.country': 'Country',
    'article.runtime': 'Runtime',
    'filter.all': 'All',
    'filter.movements': 'Movements',
    'filter.directors': 'Directors',
    'filter.films': 'Films',
    'search.placeholder': 'Search articles...',
    'tags.title': 'Tags',
    'related.title': 'Related Articles',
    'contact.title': 'Contact',
    'contact.email': 'Email',
    'contact.message': 'Message',
    'contact.submit': 'Send',
  },
} as const;

export function useTranslations(lang: Language) {
  return function t(key: keyof typeof ui[typeof defaultLang]) {
    return ui[lang][key] || ui[defaultLang][key];
  };
}

export function getLanguageFromUrl(url: URL): Language {
  const [, lang] = url.pathname.split('/');
  if (lang in languages) return lang as Language;
  return defaultLang;
}

// Route translation utilities
export const routes = {
  sk: {
    home: '/',
    about: '/sk/o-mne',
    articles: '/sk/clanky',
    contact: '/sk/kontakt',
    youtube: '/sk/youtube-eseje',
  },
  en: {
    home: '/',
    about: '/en/about',
    articles: '/en/articles',
    contact: '/en/contact',
    youtube: '/en/youtube-essays',
  },
} as const;
```

### 1.3 Tailwind Configuration (Editorial/Minimalist)

**File**: `tailwind.config.mjs`

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,ts,tsx}'],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        // Minimalist grayscale palette
        ink: {
          50: '#fafafa',
          100: '#f5f5f5',
          200: '#e5e5e5',
          300: '#d4d4d4',
          400: '#a3a3a3',
          500: '#737373',
          600: '#525252',
          700: '#404040',
          800: '#262626',
          900: '#171717',
          950: '#0a0a0a',
        },
        // Accent for links and highlights
        accent: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
        },
      },
      fontFamily: {
        sans: [
          'Inter Variable',
          'Inter',
          'system-ui',
          '-apple-system',
          'sans-serif',
        ],
        display: [
          'Space Grotesk',
          'system-ui',
          '-apple-system',
          'sans-serif',
        ],
      },
      fontSize: {
        'xs': ['0.75rem', { lineHeight: '1.5' }],
        'sm': ['0.875rem', { lineHeight: '1.6' }],
        'base': ['1rem', { lineHeight: '1.7' }],
        'lg': ['1.125rem', { lineHeight: '1.75' }],
        'xl': ['1.25rem', { lineHeight: '1.75' }],
        '2xl': ['1.5rem', { lineHeight: '1.4' }],
        '3xl': ['1.875rem', { lineHeight: '1.3' }],
        '4xl': ['2.25rem', { lineHeight: '1.2' }],
        '5xl': ['3rem', { lineHeight: '1.1' }],
        '6xl': ['3.75rem', { lineHeight: '1' }],
      },
      maxWidth: {
        'prose': '65ch',
        'wide': '80ch',
      },
      typography: (theme) => ({
        DEFAULT: {
          css: {
            '--tw-prose-body': theme('colors.ink.700'),
            '--tw-prose-headings': theme('colors.ink.900'),
            '--tw-prose-links': theme('colors.accent.600'),
            '--tw-prose-bold': theme('colors.ink.900'),
            '--tw-prose-quotes': theme('colors.ink.600'),
            '--tw-prose-code': theme('colors.ink.800'),
            maxWidth: 'none',
            lineHeight: '1.8',
            fontSize: '1.0625rem',
            a: {
              fontWeight: '500',
              textDecoration: 'underline',
              textDecorationColor: theme('colors.accent.300'),
              textUnderlineOffset: '3px',
              '&:hover': {
                textDecorationColor: theme('colors.accent.600'),
              },
            },
            'h1, h2, h3': {
              fontFamily: theme('fontFamily.display').join(', '),
              fontWeight: '600',
            },
            h1: {
              fontSize: '2.5rem',
              marginTop: '0',
              marginBottom: '1.5rem',
            },
            h2: {
              fontSize: '2rem',
              marginTop: '2.5rem',
              marginBottom: '1rem',
            },
            h3: {
              fontSize: '1.5rem',
              marginTop: '2rem',
              marginBottom: '0.75rem',
            },
            img: {
              borderRadius: '0.5rem',
              marginTop: '2rem',
              marginBottom: '2rem',
            },
            figcaption: {
              fontSize: '0.875rem',
              color: theme('colors.ink.500'),
              textAlign: 'center',
              marginTop: '1rem',
            },
            blockquote: {
              fontStyle: 'italic',
              borderLeftWidth: '3px',
              borderLeftColor: theme('colors.ink.300'),
              paddingLeft: '1.5rem',
              color: theme('colors.ink.600'),
            },
          },
        },
        dark: {
          css: {
            '--tw-prose-body': theme('colors.ink.300'),
            '--tw-prose-headings': theme('colors.ink.100'),
            '--tw-prose-links': theme('colors.accent.400'),
            '--tw-prose-bold': theme('colors.ink.100'),
            '--tw-prose-quotes': theme('colors.ink.400'),
            '--tw-prose-code': theme('colors.ink.200'),
            a: {
              textDecorationColor: theme('colors.accent.700'),
              '&:hover': {
                textDecorationColor: theme('colors.accent.400'),
              },
            },
            blockquote: {
              borderLeftColor: theme('colors.ink.700'),
              color: theme('colors.ink.400'),
            },
          },
        },
      }),
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
  ],
};
```

### 1.4 Article Card Component

**File**: `src/components/articles/ArticleCard.astro`

```astro
---
import { Image } from 'astro:assets';
import type { CollectionEntry } from 'astro:content';
import { useTranslations } from '../../lib/i18n';

interface Props {
  article: CollectionEntry<'articles'>;
  language: 'sk' | 'en';
}

const { article, language } = Astro.props;
const { slug } = article;
const { title, description, publishDate, type, coverImage } = article.data;

const t = useTranslations(language);

const articleUrl = language === 'en' 
  ? `/en/articles/${slug}`
  : `/sk/clanky/${slug}`;

const typeLabel = t(`article.type.${type}` as any);

// Reading time calculation
const content = article.body;
const wordsPerMinute = 200;
const words = content.trim().split(/\s+/).length;
const readTime = Math.ceil(words / wordsPerMinute);

// Format date
const formattedDate = publishDate.toLocaleDateString(
  language === 'sk' ? 'sk-SK' : 'en-US',
  { year: 'numeric', month: 'long', day: 'numeric' }
);
---

<article class="group">
  <a href={articleUrl} class="block">
    {coverImage && (
      <div class="aspect-[3/2] overflow-hidden rounded-lg mb-4 bg-ink-100 dark:bg-ink-800">
        <Image
          src={coverImage}
          alt={title}
          width={600}
          height={400}
          class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-105"
          loading="lazy"
        />
      </div>
    )}
    
    <div class="space-y-2">
      <div class="flex items-center gap-3 text-sm text-ink-500 dark:text-ink-400">
        <span class="px-2 py-0.5 bg-ink-100 dark:bg-ink-800 rounded-md font-medium">
          {typeLabel}
        </span>
        <time datetime={publishDate.toISOString()}>
          {formattedDate}
        </time>
        <span>·</span>
        <span>{readTime} {t('article.readTime')}</span>
      </div>

      <h2 class="text-2xl font-display font-semibold text-ink-900 dark:text-ink-100 group-hover:text-accent-600 dark:group-hover:text-accent-400 transition-colors">
        {title}
      </h2>

      <p class="text-ink-600 dark:text-ink-400 line-clamp-3">
        {description}
      </p>
    </div>
  </a>
</article>
```

### 1.5 Article Layout

**File**: `src/layouts/ArticleLayout.astro`

```astro
---
import { Image } from 'astro:assets';
import BaseLayout from './BaseLayout.astro';
import { useTranslations } from '../lib/i18n';
import type { CollectionEntry } from 'astro:content';

interface Props {
  article: CollectionEntry<'articles'>;
  language: 'sk' | 'en';
}

const { article, language } = Astro.props;
const { title, description, publishDate, type, coverImage, director, year, country, runtime } = article.data;

const t = useTranslations(language);

// Reading time
const content = article.body;
const wordsPerMinute = 200;
const words = content.trim().split(/\s+/).length;
const readTime = Math.ceil(words / wordsPerMinute);

const formattedDate = publishDate.toLocaleDateString(
  language === 'sk' ? 'sk-SK' : 'en-US',
  { year: 'numeric', month: 'long', day: 'numeric' }
);

const typeLabel = t(`article.type.${type}` as any);
---

<BaseLayout title={title} description={description} language={language}>
  <article class="max-w-prose mx-auto px-4 py-12">
    <!-- Article Header -->
    <header class="mb-12 space-y-6">
      <!-- Type badge -->
      <div class="flex items-center gap-4 text-sm">
        <span class="px-3 py-1 bg-ink-100 dark:bg-ink-800 rounded-full font-medium text-ink-700 dark:text-ink-300">
          {typeLabel}
        </span>
        <time datetime={publishDate.toISOString()} class="text-ink-500 dark:text-ink-400">
          {formattedDate}
        </time>
        <span class="text-ink-500 dark:text-ink-400">·</span>
        <span class="text-ink-500 dark:text-ink-400">
          {readTime} {t('article.readTime')}
        </span>
      </div>

      <!-- Title -->
      <h1 class="text-4xl md:text-5xl font-display font-semibold text-ink-900 dark:text-ink-100 leading-tight">
        {title}
      </h1>

      <!-- Film metadata (if type is 'film') -->
      {type === 'film' && (director || year || country || runtime) && (
        <dl class="flex flex-wrap gap-x-6 gap-y-2 text-sm text-ink-600 dark:text-ink-400 pt-4 border-t border-ink-200 dark:border-ink-700">
          {director && (
            <>
              <dt class="font-medium">{t('article.director')}:</dt>
              <dd>{director}</dd>
            </>
          )}
          {year && (
            <>
              <dt class="font-medium">{t('article.year')}:</dt>
              <dd>{year}</dd>
            </>
          )}
          {country && (
            <>
              <dt class="font-medium">{t('article.country')}:</dt>
              <dd>{country}</dd>
            </>
          )}
          {runtime && (
            <>
              <dt class="font-medium">{t('article.runtime')}:</dt>
              <dd>{runtime}</dd>
            </>
          )}
        </dl>
      )}

      <!-- Cover image -->
      {coverImage && (
        <figure class="w-full -mx-4 md:mx-0">
          <Image
            src={coverImage}
            alt={title}
            width={1200}
            height={630}
            class="w-full rounded-lg"
            loading="eager"
          />
        </figure>
      )}
    </header>

    <!-- Article Content -->
    <div class="prose prose-lg dark:prose-dark max-w-none">
      <slot />
    </div>

    <!-- Article Footer (Phase 2: Related articles, tags) -->
    <footer class="mt-16 pt-8 border-t border-ink-200 dark:border-ink-800">
      <!-- Placeholder for Phase 2 features -->
    </footer>
  </article>
</BaseLayout>
```

### 1.6 Dark Mode Toggle

**File**: `src/components/ui/DarkModeToggle.tsx`

```typescript
import { useEffect, useState } from 'react';

export default function DarkModeToggle() {
  const [isDark, setIsDark] = useState(false);

  useEffect(() => {
    // Check initial theme
    const theme = localStorage.getItem('theme');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    
    const shouldBeDark = theme === 'dark' || (!theme && prefersDark);
    setIsDark(shouldBeDark);
    
    if (shouldBeDark) {
      document.documentElement.classList.add('dark');
    }
  }, []);

  const toggleTheme = () => {
    const newTheme = !isDark;
    setIsDark(newTheme);
    
    if (newTheme) {
      document.documentElement.classList.add('dark');
      localStorage.setItem('theme', 'dark');
    } else {
      document.documentElement.classList.remove('dark');
      localStorage.setItem('theme', 'light');
    }
  };

  return (
    <button
      onClick={toggleTheme}
      aria-label="Toggle dark mode"
      className="p-2 rounded-lg hover:bg-ink-100 dark:hover:bg-ink-800 transition-colors"
    >
      {isDark ? (
        <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path
            strokeLinecap="round"
            strokeLinejoin="round"
            strokeWidth={2}
            d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
          />
        </svg>
      ) : (
        <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path
            strokeLinecap="round"
            strokeLinejoin="round"
            strokeWidth={2}
            d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z"
          />
        </svg>
      )}
    </button>
  );
}
```

### 1.7 YouTube Essays Data Structure

**File**: `src/data/youtube-essays.ts`

```typescript
export interface YouTubeEssay {
  id: string;
  title: string;
  creator: string;
  url: string;
  description?: string;
  review?: string; // Your short review
  publishDate?: Date;
  tags?: string[];
  language: 'sk' | 'en';
}

export const youtubeEssays: YouTubeEssay[] = [
  {
    id: 'example-1',
    title: 'The Beauty of Bresson\'s Cinematography',
    creator: 'Every Frame a Painting',
    url: 'https://youtube.com/watch?v=...',
    description: 'An exploration of Robert Bresson\'s minimalist visual style.',
    review: 'Insightful analysis of how Bresson uses simplicity to convey profound emotion.',
    publishDate: new Date('2023-01-15'),
    tags: ['bresson', 'cinematography', 'minimalism'],
    language: 'en',
  },
  // Add more essays...
];

// Helper functions
export function getYouTubeEssaysByLanguage(language: 'sk' | 'en'): YouTubeEssay[] {
  return youtubeEssays.filter(essay => essay.language === language);
}

export function getYouTubeVideoId(url: string): string | null {
  const match = url.match(/(?:youtube\.com\/watch\?v=|youtu\.be\/)([^&]+)/);
  return match ? match[1] : null;
}
```

### 1.8 YouTube Essays Page

**File**: `src/pages/en/youtube-essays.astro`

```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro';
import { getYouTubeEssaysByLanguage, getYouTubeVideoId } from '../../data/youtube-essays';

const essays = getYouTubeEssaysByLanguage('en');
---

<BaseLayout title="YouTube Essays" description="Curated list of film essays and video analysis" language="en">
  <div class="max-w-4xl mx-auto px-4 py-12">
    <header class="mb-12">
      <h1 class="text-4xl md:text-5xl font-display font-semibold text-ink-900 dark:text-ink-100 mb-4">
        YouTube Essays
      </h1>
      <p class="text-lg text-ink-600 dark:text-ink-400">
        A curated collection of video essays about cinema that I've found particularly insightful.
      </p>
    </header>

    <div class="space-y-8">
      {essays.map((essay) => {
        const videoId = getYouTubeVideoId(essay.url);
        return (
          <article class="border border-ink-200 dark:border-ink-800 rounded-lg overflow-hidden hover:border-accent-300 dark:hover:border-accent-700 transition-colors">
            {videoId && (
              <div class="aspect-video">
                <iframe
                  src={`https://www.youtube.com/embed/${videoId}`}
                  title={essay.title}
                  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                  allowfullscreen
                  class="w-full h-full"
                />
              </div>
            )}
            
            <div class="p-6 space-y-3">
              <div class="flex items-center gap-3 text-sm text-ink-500 dark:text-ink-400">
                <span class="font-medium">{essay.creator}</span>
                {essay.publishDate && (
                  <>
                    <span>·</span>
                    <time datetime={essay.publishDate.toISOString()}>
                      {essay.publishDate.toLocaleDateString('en-US', { year: 'numeric', month: 'short' })}
                    </time>
                  </>
                )}
              </div>

              <h2 class="text-2xl font-display font-semibold text-ink-900 dark:text-ink-100">
                <a href={essay.url} target="_blank" rel="noopener noreferrer" class="hover:text-accent-600 dark:hover:text-accent-400 transition-colors">
                  {essay.title}
                </a>
              </h2>

              {essay.description && (
                <p class="text-ink-600 dark:text-ink-400">
                  {essay.description}
                </p>
              )}

              {essay.review && (
                <blockquote class="border-l-3 border-accent-500 pl-4 text-ink-700 dark:text-ink-300 italic">
                  {essay.review}
                </blockquote>
              )}

              {essay.tags && (
                <div class="flex flex-wrap gap-2 pt-2">
                  {essay.tags.map(tag => (
                    <span class="px-2 py-1 bg-ink-100 dark:bg-ink-800 rounded text-xs font-medium text-ink-600 dark:text-ink-400">
                      #{tag}
                    </span>
                  ))}
                </div>
              )}
            </div>
          </article>
        );
      })}
    </div>
  </div>
</BaseLayout>
```

---

## Phase 2: Enhanced Discovery

### 2.1 Tag System

**Update to**: `src/content/config.ts`

```typescript
// Tags are already included in schema - no changes needed
// We just need to create tag pages

// Type for tag pages
export interface TagGroup {
  tag: string;
  count: number;
  articles: CollectionEntry<'articles'>[];
}
```

**File**: `src/pages/en/tags/[tag].astro`

```astro
---
import { getCollection } from 'astro:content';
import BaseLayout from '../../../layouts/BaseLayout.astro';
import ArticleCard from '../../../components/articles/ArticleCard.astro';

export async function getStaticPaths() {
  const articles = await getCollection('articles', ({ data }) => {
    return data.language === 'en' && !data.draft;
  });

  const tagMap = new Map<string, typeof articles>();

  articles.forEach(article => {
    if (article.data.tags) {
      article.data.tags.forEach(tag => {
        if (!tagMap.has(tag)) {
          tagMap.set(tag, []);
        }
        tagMap.get(tag)!.push(article);
      });
    }
  });

  return Array.from(tagMap.entries()).map(([tag, articles]) => ({
    params: { tag },
    props: { tag, articles },
  }));
}

const { tag, articles } = Astro.props;
---

<BaseLayout 
  title={`Articles tagged with "${tag}"`} 
  description={`Browse all articles about ${tag}`}
  language="en"
>
  <div class="max-w-6xl mx-auto px-4 py-12">
    <header class="mb-12">
      <p class="text-sm text-ink-500 dark:text-ink-400 mb-2">Tag</p>
      <h1 class="text-4xl font-display font-semibold text-ink-900 dark:text-ink-100 mb-4">
        {tag}
      </h1>
      <p class="text-ink-600 dark:text-ink-400">
        {articles.length} {articles.length === 1 ? 'article' : 'articles'}
      </p>
    </header>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      {articles.map(article => (
        <ArticleCard article={article} language="en" />
      ))}
    </div>
  </div>
</BaseLayout>
```

### 2.2 Related Articles Component

**File**: `src/lib/related-content.ts`

```typescript
import type { CollectionEntry } from 'astro:content';

export function getRelatedArticles(
  currentArticle: CollectionEntry<'articles'>,
  allArticles: CollectionEntry<'articles'>[],
  limit: number = 3
): CollectionEntry<'articles'>[] {
  const currentTags = currentArticle.data.tags || [];
  const currentType = currentArticle.data.type;

  // Filter out current article and same language
  const candidates = allArticles.filter(
    article =>
      article.slug !== currentArticle.slug &&
      article.data.language === currentArticle.data.language &&
      !article.data.draft
  );

  // Score articles based on similarity
  const scored = candidates.map(article => {
    let score = 0;

    // Same type bonus
    if (article.data.type === currentType) {
      score += 10;
    }

    // Shared tags
    const articleTags = article.data.tags || [];
    const sharedTags = currentTags.filter(tag => articleTags.includes(tag));
    score += sharedTags.length * 5;

    return { article, score };
  });

  // Sort by score and take top N
  return scored
    .sort((a, b) => b.score - a.score)
    .slice(0, limit)
    .map(item => item.article);
}
```

**File**: `src/components/articles/RelatedArticles.astro`

```astro
---
import { getRelatedArticles } from '../../lib/related-content';
import ArticleCard from './ArticleCard.astro';
import { useTranslations } from '../../lib/i18n';
import type { CollectionEntry } from 'astro:content';

interface Props {
  currentArticle: CollectionEntry<'articles'>;
  allArticles: CollectionEntry<'articles'>[];
  language: 'sk' | 'en';
}

const { currentArticle, allArticles, language } = Astro.props;
const t = useTranslations(language);

const relatedArticles = getRelatedArticles(currentArticle, allArticles, 3);
---

{relatedArticles.length > 0 && (
  <section class="mt-16 pt-8 border-t border-ink-200 dark:border-ink-800">
    <h2 class="text-2xl font-display font-semibold text-ink-900 dark:text-ink-100 mb-8">
      {t('related.title')}
    </h2>
    
    <div class="grid grid-cols-1 md:grid-cols-3 gap-8">
      {relatedArticles.map(article => (
        <ArticleCard article={article} language={language} />
      ))}
    </div>
  </section>
)}
```

### 2.3 Reading Progress Indicator

**File**: `src/components/ui/ReadingProgress.tsx`

```typescript
import { useEffect, useState } from 'react';

export default function ReadingProgress() {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const updateProgress = () => {
      const scrollTop = window.scrollY;
      const docHeight = document.documentElement.scrollHeight - window.innerHeight;
      const scrollPercent = (scrollTop / docHeight) * 100;
      setProgress(scrollPercent);
    };

    window.addEventListener('scroll', updateProgress);
    return () => window.removeEventListener('scroll', updateProgress);
  }, []);

  return (
    <div className="fixed top-0 left-0 w-full h-1 bg-ink-200 dark:bg-ink-800 z-50">
      <div
        className="h-full bg-accent-600 dark:bg-accent-400 transition-all duration-150"
        style={{ width: `${progress}%` }}
      />
    </div>
  );
}
```

### 2.4 RSS Feed

**File**: `src/pages/rss.xml.ts`

```typescript
import rss from '@astrojs/rss';
import { getCollection } from 'astro:content';
import type { APIContext } from 'astro';

export async function GET(context: APIContext) {
  const articles = await getCollection('articles', ({ data }) => !data.draft);

  // Sort by date, newest first
  const sortedArticles = articles.sort(
    (a, b) => b.data.publishDate.getTime() - a.data.publishDate.getTime()
  );

  return rss({
    title: 'Journey through Cinema',
    description: 'Essays and reviews about cinema, directors, and film movements',
    site: context.site || 'https://journeycinema.vercel.app',
    items: sortedArticles.map(article => {
      const urlPrefix = article.data.language === 'en' ? '/en/articles' : '/sk/clanky';
      return {
        title: article.data.title,
        description: article.data.description,
        pubDate: article.data.publishDate,
        link: `${urlPrefix}/${article.slug}`,
        categories: article.data.tags || [],
      };
    }),
    customData: `<language>en-us</language>`,
  });
}
```

---

## Phase 3: Search & Polish

### 3.1 Pagefind Integration

**Installation**:
```bash
npm install -D pagefind
```

**Update**: `package.json`

```json
{
  "scripts": {
    "build": "astro build && pagefind --source dist"
  }
}
```

**File**: `src/components/ui/SearchBar.tsx`

```typescript
import { useEffect, useState, useRef } from 'react';

interface SearchResult {
  id: string;
  url: string;
  title: string;
  excerpt: string;
}

export default function SearchBar({ language }: { language: 'sk' | 'en' }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<SearchResult[]>([]);
  const [isOpen, setIsOpen] = useState(false);
  const searchRef = useRef<HTMLDivElement>(null);

  const placeholder = language === 'sk' ? 'Hľadať články...' : 'Search articles...';

  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (searchRef.current && !searchRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);

  useEffect(() => {
    const performSearch = async () => {
      if (query.length < 2) {
        setResults([]);
        return;
      }

      // @ts-ignore - Pagefind is loaded globally
      if (typeof window.pagefind === 'undefined') {
        await import('/pagefind/pagefind.js');
      }

      // @ts-ignore
      const search = await window.pagefind.search(query);
      const resultData = await Promise.all(
        search.results.slice(0, 5).map((r: any) => r.data())
      );

      setResults(
        resultData.map((data: any) => ({
          id: data.url,
          url: data.url,
          title: data.meta.title,
          excerpt: data.excerpt,
        }))
      );
      setIsOpen(true);
    };

    const debounce = setTimeout(performSearch, 300);
    return () => clearTimeout(debounce);
  }, [query]);

  return (
    <div ref={searchRef} className="relative w-full max-w-md">
      <div className="relative">
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder={placeholder}
          className="w-full px-4 py-2 pl-10 border border-ink-300 dark:border-ink-700 rounded-lg
                     bg-white dark:bg-ink-900 text-ink-900 dark:text-ink-100
                     focus:ring-2 focus:ring-accent-500 focus:border-transparent
                     placeholder:text-ink-400 dark:placeholder:text-ink-600"
        />
        <svg
          className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-ink-400"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            strokeLinecap="round"
            strokeLinejoin="round"
            strokeWidth={2}
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
          />
        </svg>
      </div>

      {isOpen && results.length > 0 && (
        <div className="absolute top-full mt-2 w-full bg-white dark:bg-ink-900 border border-ink-200 dark:border-ink-800 rounded-lg shadow-lg overflow-hidden z-50">
          {results.map((result) => (
            <a
              key={result.id}
              href={result.url}
              className="block p-4 hover:bg-ink-50 dark:hover:bg-ink-800 transition-colors border-b border-ink-100 dark:border-ink-800 last:border-0"
              onClick={() => setIsOpen(false)}
            >
              <h3 className="font-semibold text-ink-900 dark:text-ink-100 mb-1">
                {result.title}
              </h3>
              <p
                className="text-sm text-ink-600 dark:text-ink-400 line-clamp-2"
                dangerouslySetInnerHTML={{ __html: result.excerpt }}
              />
            </a>
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Deployment & Configuration

### Astro Configuration

**File**: `astro.config.mjs`

```javascript
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';
import sitemap from '@astrojs/sitemap';
import mdx from '@astrojs/mdx';

export default defineConfig({
  site: 'https://journeycinema.vercel.app',
  integrations: [
    react(),
    tailwind({
      applyBaseStyles: false, // We handle this in global.css
    }),
    sitemap({
      i18n: {
        defaultLocale: 'en',
        locales: {
          en: 'en-US',
          sk: 'sk-SK',
        },
      },
    }),
    mdx(),
  ],
  output: 'static',
  build: {
    inlineStylesheets: 'auto',
  },
  markdown: {
    shikiConfig: {
      theme: 'github-dark-dimmed',
      wrap: true,
    },
  },
  vite: {
    optimizeDeps: {
      exclude: ['@resvg/resvg-js'],
    },
  },
});
```

### Environment Variables

**File**: `.env.example`

```bash
# Site Configuration
PUBLIC_SITE_URL=https://journeycinema.vercel.app

# Contact Form (optional - using Formspree)
FORMSPREE_FORM_ID=your-form-id

# Analytics (optional - Plausible)
PUBLIC_PLAUSIBLE_DOMAIN=journeycinema.vercel.app
```

### Vercel Configuration

**File**: `vercel.json`

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "astro"
}
```

---

## Content Writing Workflow

### Article Template

**File**: `src/content/articles/directors/example-director.mdx`

```mdx
---
title: "Andrei Tarkovsky: Sculpting in Time"
description: "An exploration of Tarkovsky's philosophical approach to cinema and his concept of 'sculpting in time'"
publishDate: 2024-01-15
type: "director"
language: "en"
coverImage: "./images/tarkovsky.jpg"
tags: ["russian-cinema", "philosophical-cinema", "slow-cinema"]
draft: false
---

## Introduction

Andrei Tarkovsky's cinema is an exercise in patience, contemplation, and spiritual inquiry...

## The Concept of Time

Tarkovsky viewed film as an art of "sculpting in time"...

![Scene from Mirror](./images/mirror-scene.jpg)
*Caption: A dreamlike sequence from Mirror (1975)*

## Major Works

### Solaris (1972)

A meditation on memory and human consciousness...

### Stalker (1979)

An allegorical journey through a mysterious zone...

## Conclusion

Tarkovsky's legacy continues to inspire filmmakers...
```

### Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Type checking
npm run type-check

# Linting
npm run lint

# Testing
npm run test
npm run test:e2e

# Code formatting
npm run format
```

---

## Implementation Timeline

### Phase 1: Core (Weeks 1-2)
- **Week 1, Days 1-2**: Setup, Tailwind, i18n, dark mode
- **Week 1, Days 3-5**: Content schema, article pages, layouts
- **Week 2, Days 1-2**: YouTube essays page, contact form
- **Week 2, Days 3-5**: Home page, testing, deployment

### Phase 2: Enhancement (Weeks 3-4)
- **Week 3, Days 1-2**: Tag system
- **Week 3, Days 3-5**: Related articles, reading progress
- **Week 4, Days 1-3**: RSS feed, Open Graph images
- **Week 4, Days 4-5**: Polish, refinement

### Phase 3: Search (Week 5)
- **Week 5, Days 1-2**: Pagefind integration
- **Week 5, Days 3-4**: Performance optimization
- **Week 5, Day 5**: Final testing, documentation

---

This specification provides everything needed to implement Journey through Cinema as a high-performance, content-focused website with a minimalist aesthetic perfect for long-form reading.
