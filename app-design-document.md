# SaaS Design & Development - Le Cercle des Amis Disparus

## 📋 Project Overview

**Le Cercle des Amis Disparus** est une plateforme sociale innovante qui vise à réduire l'isolement social en créant des connexions authentiques entre personnes à travers des activités géolocalisées. L'application privilégie les interactions réelles et soutient les commerces locaux.

## 🎯 Vision & Mission

### Vision
Créer un monde où personne ne se sent seul, en facilitant les rencontres authentiques et en revitalisant les liens communautaires locaux.

### Mission
- Lutter contre l'isolement social par des activités géolocalisées
- Promouvoir les commerces locaux face aux grandes surfaces
- Favoriser l'entraide et la solidarité entre citoyens
- Offrir des espaces sécurisés pour tous les types de personnalités

## 👥 Target Audience

### Personas Principaux

**1. Les Chercheurs de Connexion (25-45 ans)**
- Nouveaux arrivants dans une ville
- Personnes après une rupture/divorce
- Télétravail isolé

**2. Les Seniors Actifs (55+ ans)**
- Retraités cherchant de nouvelles activités
- Personnes veuves souhaitant recréer du lien

**3. Les Timides/Introvertis (18-65 ans)**
- Anxiété sociale mais désir de connexion
- Préfèrent les interactions structurées

**4. Les Commerçants Locaux**
- Boutiques indépendantes
- Restaurants, cafés, librairies
- Artisans et prestataires de services

## 🏗️ Technical Architecture

### Tech Stack (Next.js SaaS Starter + Extensions)

```typescript
// Core Stack - Following SaaS Starter Rules
Framework: Next.js 14+ (App Router, TypeScript strict)
Database: PostgreSQL + Drizzle ORM
Authentication: JWT + httpOnly cookies (jose library)
Payments: Stripe (webhook signature validation)
UI: shadcn/ui + Tailwind CSS + cva for variants
Deployment: Vercel
Validation: Zod for all server inputs
Logging: Activity logs table for audit trail

// Extensions Needed
Geolocation: Mapbox GL JS
Real-time: Socket.io / Pusher
Notifications: Web Push API + Email (Resend)
Image Storage: Cloudinary / AWS S3
Search: Algolia (for activities/users)
Analytics: Posthog / Mixpanel

// Development Standards
- TypeScript strict mode ("strict": true, "noEmit": true)
- kebab-case for routes, camelCase for components
- Server actions with "use server" directive
- Zod validation for all user inputs
```

### Database Schema (Drizzle ORM)

```typescript
// Following SaaS Starter patterns with soft-delete
// lib/db/schema.ts

// Core Tables
export const users = pgTable('users', {
  id: text('id').primaryKey(),
  email: text('email').unique().notNull(),
  // ... profile fields
  deletedAt: timestamp('deleted_at'), // Soft delete
});

export const activities = pgTable('activities', {
  id: text('id').primaryKey(),
  creatorId: text('creator_id').references(() => users.id),
  // ... activity fields
  deletedAt: timestamp('deleted_at'),
});

export const activityLogs = pgTable('activity_logs', {
  id: text('id').primaryKey(),
  userId: text('user_id').references(() => users.id),
  action: text('action').notNull(), // JOIN_ACTIVITY, CREATE_ACTIVITY, etc.
  timestamp: timestamp('timestamp').defaultNow(),
});

// All queries use: 
// db.select().from(users).where(isNull(users.deletedAt))
```

## 🚀 Core Features Specification

### 1. Géolocalisation Intelligente
```typescript
// app/activities/actions.ts
'use server';

import { z } from 'zod';
import { validatedActionWithUser } from '@/lib/auth/middleware';

const searchSchema = z.object({
  centerPoint: z.object({
    lat: z.number().min(-90).max(90),
    lng: z.number().min(-180).max(180)
  }),
  radius: z.number().min(1).max(50), // km
  ageRange: z.array(z.number()).length(2),
  genderPreference: z.enum(['all', 'male', 'female', 'non-binary']),
  maxParticipants: z.number().min(2).max(50),
  minParticipants: z.number().min(2)
});

export const searchActivities = validatedActionWithUser(
  searchSchema,
  async (data, user) => {
    // Safe DB query with Drizzle
    return await db.select()
      .from(activities)
      .where(
        and(
          isNull(activities.deletedAt),
          // Geographic bounds logic
        )
      )
      .limit(20);
  }
);
```

### 2. Chat Intégré Temps Réel
```typescript
// lib/chat/validation.ts
export const messageSchema = z.object({
  content: z.string().min(1).max(500),
  activityId: z.string(),
  type: z.enum(['text', 'location'])
});

// Server action for message creation
export const sendMessage = validatedActionWithUser(
  messageSchema,
  async (data, user) => {
    // Log activity
    await db.insert(activityLogs).values({
      userId: user.id,
      action: 'SEND_MESSAGE',
      metadata: { activityId: data.activityId }
    });
    
    // Insert message with proper validation
    return await db.insert(messages).values({
      ...data,
      userId: user.id,
      timestamp: new Date()
    }).returning();
  }
);
```

### 3. Système de Bons Plans
```javascript
// Merchant deals integration
const dealSystem = {
  autoTrigger: {
    participantThreshold: number,
    autoBook: boolean,
    notifyMerchant: boolean
  },
  sponsorship: {
    merchantCanSponsor: boolean,
    targetDemographics: object
  }
}
```

### 4. Module "Les Invisibles"
```javascript
// Safe space for shy users
const invisibleSpace = {
  features: ['read', 'listen', 'write'],
  anonymity: 'optional',
  moderation: 'strict',
  graduated_exposure: true // Progressive comfort levels
}
```

### 5. Système de Cadeaux
```javascript
// Anonymous gifting system
const giftSystem = {
  anonymous: boolean,
  merchantSponsored: boolean,
  categories: ['coffee', 'meal', 'activity', 'transport'],
  redemptionProcess: 'qr_code' | 'code'
}
```

## 🔐 Security & Privacy

### Security & Privacy (Following SaaS Starter Rules)

```typescript
// lib/auth/session.ts - Following starter patterns
import { SignJWT, jwtVerify } from 'jose';

export async function createSession(userId: string) {
  const token = await new SignJWT({ userId })
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('24h')
    .sign(secret);
    
  cookies().set('session', token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'lax',
    maxAge: 24 * 60 * 60 // 24 hours
  });
}

// Webhook signature verification (Stripe)
export async function POST(request: Request) {
  const signature = request.headers.get('stripe-signature');
  
  try {
    const event = stripe.webhooks.constructEvent(
      body, signature, process.env.STRIPE_WEBHOOK_SECRET!
    );
    
    // Process webhook with proper logging
    await db.insert(activityLogs).values({
      action: 'STRIPE_WEBHOOK',
      metadata: { eventType: event.type }
    });
    
  } catch (err) {
    console.error('Webhook signature verification failed:', err);
    return Response.json({ error: 'Invalid signature' }, { status: 400 });
  }
}
```

### Data Protection (GDPR Compliant)
- Soft delete implementation: `deletedAt` field in all tables
- User data export via server actions
- Cookie consent with proper httpOnly settings
- Activity logging for audit trail
- Environment variables for all secrets

## 💰 Business Model

### Revenue Streams

**1. Commission sur Réservations (5-10%)**
- Commission sur les réservations automatiques
- Partenariats avec commerces locaux

**2. Abonnements Premium (€9.99/mois)**
- Création d'activités illimitées
- Priorité dans les matchings
- Statistiques détaillées
- Badge "Organisateur Vérifié"

**3. Publicité Contextuelle**
- Commerces locaux uniquement
- Géotargeting précis
- Non-intrusive

**4. Services aux Entreprises (B2B)**
- Team building géolocalisé
- Events corporate
- CSR packages

## 📱 User Experience Design

### Design Principles
- **Simplicité**: Interface claire, onboarding fluide
- **Bienveillance**: Ton chaleureux, pas de compétition
- **Inclusivité**: Accessible à tous les âges et capacités
- **Authenticité**: Favorise les vraies rencontres

### Mobile-First Approach
```css
/* Progressive Web App */
Offline-capable: Basic functionality without internet
Push notifications: Gentle reminders and updates
Native-like: Install on home screen
Fast loading: < 3 seconds initial load
```

### Accessibility
- WCAG 2.1 AA compliant
- Screen reader optimized
- High contrast mode
- Large text options
- Voice navigation support

## 🔄 User Journey

### New User Onboarding
1. **Welcome** - Explanation de la mission
2. **Location Permission** - Avec explications claires
3. **Profile Setup** - Minimal, extensible later
4. **Preferences** - Age, interests, mobility
5. **First Discovery** - Suggested activities nearby
6. **Safety Brief** - Guidelines et outils de sécurité

### Activity Creation Flow
1. **Activity Type** - Prédéfini ou libre
2. **Location & Radius** - Carte interactive
3. **Requirements** - Age, gender, group size
4. **Timing** - Date, durée, flexibilité
5. **Description** - Auto-suggestions bienveillantes
6. **Preview & Publish** - Vérification finale

### Participation Flow
1. **Discovery** - Carte ou liste
2. **Activity Details** - Toutes infos transparentes
3. **Join Request** - Simple click
4. **Approval** - Auto ou manual par créateur
5. **Pre-Meeting Chat** - Coordination
6. **Activity Reminder** - 24h, 2h avant
7. **Post-Activity** - Feedback optionnel

## 🧠 AI & Machine Learning

### Smart Matching Algorithm
```python
# Matching factors (weighted)
matching_score = {
  'geographic_proximity': 0.3,
  'age_compatibility': 0.2,
  'interest_overlap': 0.2,
  'personality_fit': 0.15,
  'availability_match': 0.15
}
```

### Content Moderation
- Auto-detection de contenu inapproprié
- Classification des activités par sécurité
- Détection de signaux de détresse
- Suggestions d'amélioration de profils

### Recommendation Engine
- Activités similaires à celles aimées
- Nouveaux types d'activités à essayer
- Moments optimaux pour créer des événements
- Commerces partenaires pertinents

## 📊 Analytics & KPIs

### User Engagement
- Activités créées par mois
- Taux de participation aux activités
- Durée moyenne des conversations
- Retention rate (1 jour, 7 jours, 30 jours)

### Social Impact
- Nombre de nouvelles connexions créées
- Réduction de l'isolement (survey-based)
- Activités solidaires organisées
- Montant des cadeaux offerts

### Business Metrics
- Revenue per user (RPU)
- Customer acquisition cost (CAC)
- Merchant conversion rate
- Commission revenue growth

## 🚀 Development Roadmap

### Phase 1 - MVP (3 mois)
- [ ] Authentification et profils de base
- [ ] Création/recherche d'activités géolocalisées
- [ ] Chat de groupe basique
- [ ] Interface commerces partenaires
- [ ] Système de réservation simple

### Phase 2 - Social Features (2 mois)
- [ ] Module "Les Invisibles"
- [ ] Système de cadeaux
- [ ] Notifications push intelligentes
- [ ] Activités "phone-free"
- [ ] Amélioration UX mobile

### Phase 3 - Business & Scale (2 mois)
- [ ] Tableau de bord analytics
- [ ] Système d'abonnement premium
- [ ] API publique pour partenaires
- [ ] Modération IA avancée
- [ ] Internationalisation

### Phase 4 - Advanced Features (3 mois)
- [ ] Matching algorithm ML
- [ ] Réalité augmentée pour découverte
- [ ] Intégration calendriers externes
- [ ] Système de réputation
- [ ] Programme d'ambassadeurs

## 🔧 Development Guidelines

### Code Standards (SaaS Starter Compliance)
```typescript
// tsconfig.json requirements
{
  "compilerOptions": {
    "strict": true,
    "noEmit": true
  }
}

// File naming conventions
/app/activities/create-activity/page.tsx (kebab-case routes)
/components/ui/ActivityCard.tsx (PascalCase components)  
/lib/activities/validation.ts (camelCase utilities)

// Component patterns with proper typing
interface ActivityCardProps extends React.ComponentProps<'div'> {
  activity: Activity;
  onJoin: (id: string) => Promise<void>;
}

export function ActivityCard({ activity, onJoin, ...props }: ActivityCardProps) {
  return (
    <div {...props} className={cn('card', props.className)}>
      {/* Component content */}
    </div>
  );
}

// Server actions with validation
export const createActivity = validatedActionWithUser(
  createActivitySchema,
  async (data, user) => {
    // Business logic here
    const result = await db.insert(activities)
      .values({ ...data, creatorId: user.id })
      .returning();
    
    // Audit log
    await db.insert(activityLogs).values({
      userId: user.id,
      action: 'CREATE_ACTIVITY',
      metadata: { activityId: result[0].id }
    });
    
    return result[0];
  }
);
```

### Performance Targets
- First Contentful Paint: < 1.5s
- Largest Contentful Paint: < 2.5s
- Cumulative Layout Shift: < 0.1
- First Input Delay: < 100ms

### Monitoring & Observability
- Error tracking: Sentry
- Performance: Web Vitals
- User behavior: Hotjar/LogRocket
- Uptime monitoring: Pingdom
- Real-time alerts: PagerDuty

## 🌍 Scaling Considerations

### Technical Scaling
- Database sharding by geography
- CDN for static assets
- Caching strategy (Redis)
- Microservices architecture (future)
- Multi-region deployment

### Geographic Expansion
1. **Phase 1**: Major French cities
2. **Phase 2**: French-speaking regions
3. **Phase 3**: European markets
4. **Phase 4**: Global expansion

### Team Structure
```
Product Team: PM + UX Designer + Researcher
Engineering: 2-3 Fullstack + 1 Mobile + DevOps
Growth: Marketing + Community Manager
Operations: Customer Support + Data Analyst
```

## 🎨 Brand & Design System

### Visual Identity
- **Colors**: Warm, welcoming palette
- **Typography**: Accessible, friendly fonts
- **Icons**: Consistent, inclusive iconography
- **Illustrations**: Diverse, authentic representation

### Tone of Voice
- Bienveillant et encourageant
- Simple sans être infantilisant  
- Inclusif et respectueux
- Optimiste mais réaliste

---

## 🏁 Success Criteria

### Year 1 Goals
- 10,000+ active users
- 500+ activities per month
- 100+ merchant partners
- 85%+ user satisfaction score
- Break-even on operational costs

### Long-term Vision (5 ans)
- Présence dans 50+ villes françaises
- 500,000+ utilisateurs actifs
- Impact social mesurable sur l'isolement
- Écosystème auto-suffisant
- Modèle exportable internationalement

---

*Ce document doit être mis à jour régulièrement selon les feedbacks utilisateurs et l'évolution du produit.*