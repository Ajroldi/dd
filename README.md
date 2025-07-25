# REPORT COMPLETO - APPLICAZIONE LINKSHORT

## INDICE
1. [Panoramica Generale](#panoramica-generale)
2. [Architettura dell'Applicazione](#architettura-dellapplicazione)
3. [Sistema di Autenticazione](#sistema-di-autenticazione)
4. [Gestione Dati e Storage](#gestione-dati-e-storage)
5. [Funzionalità Principali](#funzionalità-principali)
6. [Sistema di Redirect](#sistema-di-redirect)
7. [Sistema Analytics](#sistema-analytics)
8. [Navigazione e Routing](#navigazione-e-routing)
9. [Gestione Stato e Performance](#gestione-stato-e-performance)
10. [Sicurezza e Validazione](#sicurezza-e-validazione)
11. [Deployment e Build](#deployment-e-build)
12. [Limitazioni Attuali](#limitazioni-attuali)
13. [Possibili Miglioramenti](#possibili-miglioramenti)

---

## PANORAMICA GENERALE

**LinkShort** è un'applicazione web moderna per l'accorciamento di URL, sviluppata con React e TypeScript. L'applicazione offre funzionalità complete simili a servizi come bit.ly o tinyurl, permettendo agli utenti di:

- **Registrarsi e autenticarsi** nel sistema
- **Creare link abbreviati** da URL lunghi
- **Personalizzare codici** per i link abbreviati
- **Monitorare statistiche dettagliate** dei click
- **Gestire scadenze** dei link
- **Visualizzare analytics avanzate** con grafici

### Caratteristiche Distintive
- **Interfaccia moderna** con design responsive
- **Gestione completa del ciclo di vita** dei link
- **Analytics dettagliate** con visualizzazioni grafiche
- **Codici personalizzabili** per branding
- **Sistema di scadenza** automatica
- **Tracking completo** dei click

---

## ARCHITETTURA DELL'APPLICAZIONE

### Stack Tecnologico Utilizzato

**Frontend Framework:**
- **React 18**: Libreria principale per UI
- **TypeScript**: Tipizzazione statica per maggiore robustezza
- **React Router DOM v7**: Gestione routing e navigazione

**Styling e UI:**
- **Tailwind CSS**: Framework CSS utility-first
- **Lucide React**: Libreria icone moderne e leggere
- **Design System**: Palette colori coerente e componenti riutilizzabili

**Visualizzazione Dati:**
- **Recharts**: Libreria per grafici e visualizzazioni
- **Date-fns**: Manipolazione e formattazione date

**Build e Development:**
- **Vite**: Build tool moderno e veloce
- **ESLint**: Linting del codice
- **PostCSS + Autoprefixer**: Processamento CSS

### Struttura Organizzativa del Codice

```
src/
├── components/          # Componenti riutilizzabili
│   ├── CreateLinkForm.tsx    # Form creazione link
│   ├── LinkCard.tsx          # Card visualizzazione link
│   └── Navbar.tsx            # Barra navigazione
├── contexts/           # Context API per stato globale
│   └── AuthContext.tsx       # Gestione autenticazione
├── pages/              # Pagine principali dell'app
│   ├── Dashboard.tsx         # Pagina principale utente
│   ├── Analytics.tsx         # Statistiche e grafici
│   ├── Login.tsx            # Autenticazione
│   └── Redirect.tsx         # Gestione redirect link
├── types/              # Definizioni TypeScript
│   └── index.ts             # Interfacce principali
├── utils/              # Funzioni di utilità
│   └── storage.ts           # Gestione localStorage
└── main.tsx           # Entry point applicazione
```

### Principi Architetturali Seguiti

**Separation of Concerns:**
- Componenti UI separati dalla logica business
- Context per stato globale isolato
- Utilities per funzioni riutilizzabili

**Component-Based Architecture:**
- Componenti piccoli e focalizzati
- Props tipizzate con TypeScript
- Riutilizzabilità massimizzata

**Unidirectional Data Flow:**
- Stato fluisce dall'alto verso il basso
- Eventi risalgono attraverso callback
- Context per stato condiviso

---

## SISTEMA DI AUTENTICAZIONE

### AuthContext - Gestione Stato Globale

Il file `src/contexts/AuthContext.tsx` implementa un sistema di autenticazione completo usando React Context API.

**Interfaccia AuthContextType:**
```typescript
interface AuthContextType {
  user: User | null;           // Utente corrente o null
  login: (email: string, password: string) => Promise<boolean>;
  register: (name: string, email: string, password: string) => Promise<boolean>;
  logout: () => void;
  isLoading: boolean;          // Stato caricamento iniziale
}
```

**Funzionalità Implementate:**

1. **Inizializzazione:**
   - All'avvio controlla localStorage per utente esistente
   - Imposta stato loading durante verifica
   - Ripristina sessione se valida

2. **Processo di Login:**
   - Ricerca utente per email nel localStorage
   - Se trovato, imposta come utente corrente
   - Salva stato in localStorage per persistenza
   - Ritorna boolean per successo/fallimento

3. **Processo di Registrazione:**
   - Verifica unicità email
   - Crea nuovo oggetto User con ID timestamp
   - Aggiunge a collezione utenti
   - Auto-login dopo registrazione

4. **Logout:**
   - Pulisce stato utente corrente
   - Rimuove dati da localStorage
   - Redirect automatico a login

### Pagina Login - Interfaccia Utente

Il componente `src/pages/Login.tsx` fornisce un'interfaccia unificata per login e registrazione.

**Caratteristiche UI:**
- **Toggle dinamico** tra modalità login/registrazione
- **Form validation** in tempo reale
- **Error handling** con messaggi specifici
- **Loading states** durante operazioni async
- **Design responsive** ottimizzato mobile

**Gestione Form:**
```typescript
const [formData, setFormData] = useState({
  name: '',      // Solo per registrazione
  email: '',     // Richiesto sempre
  password: '',  // Richiesto sempre
});
```

**Validazioni Client-Side:**
- Email formato valido
- Password non vuota
- Nome richiesto per registrazione
- Feedback immediato errori

---

## GESTIONE DATI E STORAGE

### Sistema Storage Simulato

Il file `src/utils/storage.ts` implementa un sistema di persistenza usando localStorage che simula un database relazionale.

**Collezioni Dati:**
- `url_shortener_users`: Array di tutti gli utenti registrati
- `url_shortener_links`: Array di tutti i link creati
- `url_shortener_current_user`: Utente attualmente autenticato

**Funzioni Core:**

1. **generateShortCode():**
   ```typescript
   // Genera codici casuali di 4-6 caratteri
   const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
   const length = Math.floor(Math.random() * 3) + 4; // 4-6 caratteri
   ```

2. **isShortCodeAvailable():**
   - Verifica unicità codice tra tutti i link
   - Controlla sia shortCode che customCode
   - Previene duplicazioni

3. **validateUrl():**
   ```typescript
   try {
     new URL(url);  // Usa costruttore nativo per validazione
     return true;
   } catch {
     return false;
   }
   ```

### Strutture Dati TypeScript

**User Interface:**
```typescript
interface User {
  id: string;           // Timestamp di creazione come stringa
  email: string;        // Email univoca per login
  name: string;         // Nome display nell'interfaccia
  createdAt: string;    // Data registrazione in formato ISO
}
```

**Link Interface:**
```typescript
interface Link {
  id: string;           // ID univoco (timestamp)
  userId: string;       // Riferimento al proprietario
  originalUrl: string;  // URL completo originale
  shortCode: string;    // Codice generato automaticamente
  customCode?: string;  // Codice personalizzato opzionale
  title?: string;       // Titolo descrittivo opzionale
  createdAt: string;    // Data creazione ISO
  expiresAt?: string;   // Data scadenza opzionale ISO
  isActive: boolean;    // Flag attivo/disattivo
  clicks: Click[];      // Array di tutti i click ricevuti
}
```

**Click Interface:**
```typescript
interface Click {
  id: string;           // ID univoco del click
  linkId: string;       // Riferimento al link cliccato
  timestamp: string;    // Momento esatto del click (ISO)
  userAgent?: string;   // Browser/device dell'utente
  ipAddress?: string;   // IP address (simulato)
}
```

---

## FUNZIONALITÀ PRINCIPALI

### Dashboard - Pagina Principale

Il componente `src/pages/Dashboard.tsx` è il cuore dell'applicazione, dove gli utenti gestiscono i loro link.

**Layout Responsive:**
- **Colonna sinistra (1/3)**: Form creazione nuovo link (toggle)
- **Colonna destra (2/3)**: Lista link esistenti con funzionalità

**Funzionalità Implementate:**

1. **Gestione Link:**
   - Caricamento automatico link utente
   - Ordinamento cronologico (più recenti primi)
   - Filtraggio per utente corrente

2. **Ricerca Avanzata:**
   ```typescript
   const filteredLinks = links.filter(link => 
     link.originalUrl.toLowerCase().includes(searchTerm.toLowerCase()) ||
     link.title?.toLowerCase().includes(searchTerm.toLowerCase()) ||
     link.shortCode.toLowerCase().includes(searchTerm.toLowerCase()) ||
     link.customCode?.toLowerCase().includes(searchTerm.toLowerCase())
   );
   ```

3. **Statistiche Rapide:**
   - Contatore totale link
   - Informazioni pubbliche sui link
   - Esempi URL per comprensione

### Form Creazione Link

Il componente `src/components/CreateLinkForm.tsx` gestisce la creazione di nuovi link abbreviati.

**Campi del Form:**

1. **URL Originale (Obbligatorio):**
   - Validazione formato URL
   - Supporto protocolli http/https
   - Trim automatico spazi

2. **Titolo (Opzionale):**
   - Descrizione personalizzata
   - Migliora organizzazione link
   - Usato in ricerca e visualizzazione

3. **Codice Personalizzato (Opzionale):**
   - Alternativa al codice auto-generato
   - Validazione: min 3 caratteri
   - Solo alfanumerici, trattini, underscore
   - Verifica unicità in tempo reale

4. **Data Scadenza (Opzionale):**
   - DateTime picker nativo
   - Validazione data futura
   - Disattivazione automatica alla scadenza

**Processo di Validazione:**
```typescript
const validateForm = () => {
  const newErrors: Record<string, string> = {};
  
  // Validazione URL
  if (!originalUrl.trim()) {
    newErrors.originalUrl = 'URL richiesto';
  } else if (!validateUrl(originalUrl)) {
    newErrors.originalUrl = 'URL non valido';
  }
  
  // Validazione codice personalizzato
  if (customCode && customCode.trim().length < 3) {
    newErrors.customCode = 'Minimo 3 caratteri';
  }
  
  // Pattern validation
  if (customCode && !/^[a-zA-Z0-9-_]+$/.test(customCode.trim())) {
    newErrors.customCode = 'Solo lettere, numeri, - e _';
  }
  
  // Unicità codice
  if (customCode && !isShortCodeAvailable(customCode.trim())) {
    newErrors.customCode = 'Codice già in uso';
  }
  
  return Object.keys(newErrors).length === 0;
};
```

### Visualizzazione Link

Il componente `src/components/LinkCard.tsx` mostra ogni link in una card informativa.

**Informazioni Visualizzate:**
- **Header**: Titolo o "Link senza titolo"
- **URL Originale**: Troncato per spazio
- **URL Abbreviato**: Completo e copiabile
- **Statistiche**: Numero click totali
- **Date**: Creazione e scadenza
- **Stato**: Attivo/Scaduto con indicatori visivi

**Azioni Disponibili:**

1. **Copia URL:**
   ```typescript
   const copyToClipboard = async () => {
     try {
       await navigator.clipboard.writeText(shortUrl);
       setCopied(true);
       setTimeout(() => setCopied(false), 2000);
     } catch (err) {
       console.error('Failed to copy:', err);
     }
   };
   ```

2. **Apri URL Originale:**
   - Nuova tab con window.open()
   - Preserva contesto applicazione

3. **Elimina Link:**
   - Rimozione permanente da storage
   - Aggiornamento immediato UI

**Stati Visivi:**
- **Link Attivi**: Bordo grigio, sfondo bianco
- **Link Scaduti**: Bordo rosso, sfondo rosso chiaro
- **Feedback Copia**: Messaggio temporaneo "Copiato!"

---

## SISTEMA DI REDIRECT

### Gestione Redirect Pubblici

Il componente `src/pages/Redirect.tsx` gestisce il redirect dei link abbreviati, funzionando come endpoint pubblico.

**Flusso Operativo Completo:**

1. **Estrazione Parametri:**
   ```typescript
   const { code } = useParams<{ code: string }>();
   ```

2. **Ricerca Link:**
   ```typescript
   const link = links.find(l => 
     (l.shortCode === code || l.customCode === code) && l.isActive
   );
   ```

3. **Validazioni Multiple:**
   - Link deve esistere nel database
   - Link deve essere attivo (isActive: true)
   - Link non deve essere scaduto
   - Codice deve corrispondere (shortCode O customCode)

4. **Controllo Scadenza:**
   ```typescript
   if (link.expiresAt && isAfter(new Date(), new Date(link.expiresAt))) {
     window.location.href = '/';  // Redirect a home se scaduto
     return;
   }
   ```

5. **Registrazione Click:**
   ```typescript
   const newClick: Click = {
     id: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
     linkId: link.id,
     timestamp: new Date().toISOString(),
     userAgent: navigator.userAgent,
     ipAddress: 'client-ip',  // Simulato
   };
   ```

6. **Aggiornamento Storage:**
   - Aggiunge click all'array del link
   - Salva modifiche in localStorage
   - Mantiene integrità dati

7. **Redirect Finale:**
   ```typescript
   window.location.href = link.originalUrl;
   ```

**Gestione Errori:**
- **Codice mancante**: Redirect automatico alla home
- **Link non trovato**: Redirect automatico alla home
- **Link scaduto**: Redirect automatico alla home
- **Link disattivo**: Redirect automatico alla home

**Ottimizzazioni Performance:**
- Prevenzione esecuzioni multiple con flag
- Redirect immediato senza delay
- UI minimale durante elaborazione

---

## SISTEMA ANALYTICS

### Dashboard Statistiche

Il componente `src/pages/Analytics.tsx` fornisce una dashboard completa per l'analisi delle performance dei link.

**Metriche Principali Calcolate:**

1. **Click Totali:**
   ```typescript
   const getTotalClicks = () => {
     const filteredLinks = selectedLink === 'all' ? links : links.filter(link => link.id === selectedLink);
     return filteredLinks.reduce((total, link) => total + link.clicks.length, 0);
   };
   ```

2. **Link Attivi:**
   ```typescript
   const getActiveLinks = () => {
     return filteredLinks.filter(link => {
       if (!link.expiresAt) return true;
       return new Date(link.expiresAt) > new Date();
     }).length;
   };
   ```

3. **Media Click per Link:**
   - Calcolo automatico: totale click / numero link
   - Arrotondamento per leggibilità

**Sistema di Filtri:**
- **Dropdown Selezione**: Tutti i link o link specifico
- **Aggiornamento Dinamico**: Tutte le metriche si aggiornano automaticamente
- **Persistenza Selezione**: Mantiene filtro durante navigazione

### Visualizzazioni Grafiche

**Grafico Click Giornalieri:**
```typescript
const generateDailyClickData = () => {
  const days = [];
  
  // Genera array ultimi 7 giorni
  for (let i = 6; i >= 0; i--) {
    const date = subDays(new Date(), i);
    days.push({
      date: format(date, 'MMM d'),
      clicks: 0,
    });
  }
  
  // Conta click per ogni giorno
  filteredLinks.forEach(link => {
    link.clicks.forEach(click => {
      const clickDate = new Date(click.timestamp);
      days.forEach(day => {
        const dayDate = subDays(new Date(), 6 - days.indexOf(day));
        if (isWithinInterval(clickDate, {
          start: startOfDay(dayDate),
          end: endOfDay(dayDate)
        })) {
          day.clicks++;
        }
      });
    });
  });
  
  return days;
};
```

**Componente Grafico Recharts:**
```typescript
<ResponsiveContainer width="100%" height="100%">
  <BarChart data={dailyData}>
    <CartesianGrid strokeDasharray="3 3" />
    <XAxis dataKey="date" />
    <YAxis />
    <Tooltip />
    <Bar dataKey="clicks" fill="#3B82F6" />
  </BarChart>
</ResponsiveContainer>
```

**Top Links Performance:**
```typescript
const getTopPerformingLinks = () => {
  return filteredLinks
    .sort((a, b) => b.clicks.length - a.clicks.length)
    .slice(0, 5);
};
```

### Manipolazione Date Avanzata

**Libreria date-fns utilizzata per:**
- `subDays()`: Calcolo giorni precedenti
- `format()`: Formattazione date per display
- `isWithinInterval()`: Verifica appartenenza range temporale
- `startOfDay()` / `endOfDay()`: Definizione boundaries giornaliere

---

## NAVIGAZIONE E ROUTING

### Configurazione Router

Il componente principale `src/App.tsx` implementa un sistema di routing completo con protezione delle route.

**Struttura Route:**
```typescript
<Routes>
  <Route 
    path="/login" 
    element={user ? <Navigate to="/" /> : <Login />} 
  />
  <Route 
    path="/" 
    element={
      <ProtectedRoute>
        <Dashboard />
      </ProtectedRoute>
    } 
  />
  <Route 
    path="/analytics" 
    element={
      <ProtectedRoute>
        <Analytics />
      </ProtectedRoute>
    } 
  />
  {/* Route pubblica per redirect - DEVE essere ultima */}
  <Route path="/:code" element={<Redirect />} />
</Routes>
```

**Componente ProtectedRoute:**
```typescript
const ProtectedRoute: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const { user, isLoading } = useAuth();
  
  if (isLoading) {
    return <LoadingSpinner />;
  }
  
  return user ? <>{children}</> : <Navigate to="/login" />;
};
```

**Logica di Redirect:**
- Utenti non autenticati → `/login`
- Utenti autenticati su `/login` → `/` (Dashboard)
- Route protette richiedono autenticazione
- Route pubblica `/:code` per redirect link

### Navbar - Navigazione Principale

Il componente `src/components/Navbar.tsx` fornisce navigazione per utenti autenticati.

**Elementi Navbar:**
- **Logo**: LinkShort con icona Link2
- **Menu Principale**: Dashboard e Analytics
- **Indicatori Attivi**: Highlight route corrente
- **Profilo Utente**: Nome e logout
- **Design Responsive**: Ottimizzato mobile

**Gestione Stati Attivi:**
```typescript
const isActive = (path: string) => location.pathname === path;

// Applicazione classi condizionali
className={`px-3 py-2 rounded text-sm font-medium ${
  isActive('/') 
    ? 'bg-blue-100 text-blue-700' 
    : 'text-gray-600 hover:text-gray-900'
}`}
```

---

## GESTIONE STATO E PERFORMANCE

### Pattern Context API

**Vantaggi dell'approccio scelto:**
- **Stato Globale Centralizzato**: AuthContext per autenticazione
- **Prop Drilling Evitato**: Accesso diretto tramite useAuth()
- **Type Safety**: Interfacce TypeScript per tutto lo stato
- **Performance**: Context specifici per domini diversi

**Hook Personalizzato:**
```typescript
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### Ottimizzazioni Performance

**Gestione Re-render:**
- **Local State**: Stato UI temporaneo nei componenti
- **Memoization**: Calcoli costosi cachati
- **Effect Dependencies**: useEffect ottimizzati

**Esempio Ottimizzazione:**
```typescript
// Dashboard - caricamento link ottimizzato
useEffect(() => {
  if (user) {
    const allLinks = getLinks();
    const userLinks = allLinks.filter(link => link.userId === user.id);
    setLinks(userLinks.sort((a, b) => 
      new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
    ));
  }
}, [user]); // Dipendenza specifica
```

**Lazy Loading Potenziale:**
- Componenti caricabili on-demand
- Code splitting per route
- Dynamic imports per librerie pesanti

---

## SICUREZZA E VALIDAZIONE

### Validazioni Client-Side

**URL Validation:**
```typescript
export const validateUrl = (url: string): boolean => {
  try {
    new URL(url);  // Costruttore nativo JavaScript
    return true;
  } catch {
    return false;
  }
};
```

**Codice Personalizzato:**
```typescript
// Pattern regex per caratteri permessi
if (customCode && !/^[a-zA-Z0-9-_]+$/.test(customCode.trim())) {
  newErrors.customCode = 'Solo lettere, numeri, - e _';
}
```

**Input Sanitization:**
- `trim()` automatico su tutti gli input
- Controlli lunghezza minima/massima
- Escape caratteri speciali dove necessario

### Gestione Errori Robusta

**Try-Catch Blocks:**
```typescript
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();
  setIsSubmitting(true);
  
  try {
    // Operazioni critiche
    const success = await login(formData.email, formData.password);
    if (!success) {
      setError('Credenziali non valide');
    }
  } catch (error) {
    setError('Errore di sistema. Riprova.');
  } finally {
    setIsSubmitting(false);
  }
};
```

**Error Boundaries Potenziali:**
- Cattura errori React non gestiti
- Fallback UI per errori critici
- Logging errori per debugging

### Limitazioni Sicurezza Attuali

**Client-Side Only:**
- Validazioni bypassabili
- Dati esposti in localStorage
- Nessuna autenticazione server-side

**Mancanza Rate Limiting:**
- Possibili abusi creazione link
- Spam click non prevenibile
- Nessun throttling richieste

---

## DEPLOYMENT E BUILD

### Configurazione Vite

Il file `vite.config.ts` configura il build tool:

```typescript
export default defineConfig({
  plugins: [react()],
  optimizeDeps: {
    exclude: ['lucide-react'],  // Evita pre-bundling per performance
  },
});
```

**Vantaggi Vite:**
- **Hot Module Replacement**: Aggiornamenti istantanei
- **Fast Refresh**: Preserva stato componenti
- **ES Modules**: Build ottimizzati
- **Tree Shaking**: Rimozione codice inutilizzato

### Configurazione Netlify

**File `public/_redirects`:**
```
/*    /index.html   200
```

**Funzionalità:**
- **SPA Routing**: Tutte le route servono index.html
- **Client-Side Routing**: React Router gestisce navigazione
- **Fallback**: 404 → index.html per route dinamiche

**Build Process:**
1. `npm run build` → Genera cartella `dist/`
2. Ottimizzazione assets automatica
3. Minificazione CSS/JS
4. Deploy automatico su push

### Ottimizzazioni Build

**Bundle Splitting:**
- Vendor chunks separati
- Route-based code splitting potenziale
- Asset optimization automatica

**Performance Metrics:**
- Lighthouse score ottimizzabile
- Core Web Vitals monitorabili
- Bundle size analysis disponibile

---

## LIMITAZIONI ATTUALI

### Storage e Persistenza

**LocalStorage Limitations:**
- **Capacità**: ~5-10MB limite browser
- **Persistenza**: Solo locale, non cross-device
- **Concorrenza**: Nessuna gestione conflitti
- **Backup**: Nessun sistema di backup automatico

**Problemi Scalabilità:**
```typescript
// Tutti i dati caricati in memoria
const allLinks = getLinks(); // Può diventare lento con molti link
```

### Sicurezza e Autenticazione

**Mancanze Critiche:**
- **No Backend Validation**: Tutte le validazioni bypassabili
- **No Encryption**: Dati in chiaro in localStorage
- **No Session Management**: Nessuna scadenza token
- **No Rate Limiting**: Abusi possibili

**Vulnerabilità Potenziali:**
- XSS attraverso URL malformati
- CSRF su operazioni critiche
- Data tampering in localStorage

### Performance e UX

**Problemi Performance:**
- **Linear Search**: O(n) per ricerca link
- **No Pagination**: Tutti i link caricati insieme
- **No Caching**: Calcoli analytics ripetuti
- **Memory Leaks**: Possibili con molti componenti

**Limitazioni UX:**
- **No Offline Support**: Richiede connessione
- **No Real-time Updates**: Nessun sync multi-device
- **No Bulk Operations**: Una operazione alla volta

---

## POSSIBILI MIGLIORAMENTI

### Backend Integration

**Database Reale:**
```sql
-- Schema PostgreSQL suggerito
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE links (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  original_url TEXT NOT NULL,
  short_code VARCHAR(10) UNIQUE NOT NULL,
  custom_code VARCHAR(50) UNIQUE,
  title VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP,
  is_active BOOLEAN DEFAULT true
);

CREATE TABLE clicks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  link_id UUID REFERENCES links(id),
  timestamp TIMESTAMP DEFAULT NOW(),
  user_agent TEXT,
  ip_address INET,
  country VARCHAR(2),
  city VARCHAR(100)
);
```

**API REST Endpoints:**
```typescript
// Struttura API suggerita
POST /api/auth/login
POST /api/auth/register
POST /api/auth/logout
GET  /api/links
POST /api/links
PUT  /api/links/:id
DELETE /api/links/:id
GET  /api/analytics/:linkId
GET  /api/redirect/:code
```

### Sicurezza Avanzata

**JWT Authentication:**
```typescript
// Token structure
interface JWTPayload {
  userId: string;
  email: string;
  exp: number;
  iat: number;
}

// Refresh token rotation
interface AuthTokens {
  accessToken: string;   // 15 minuti
  refreshToken: string;  // 7 giorni
}
```

**Rate Limiting:**
```typescript
// Express middleware esempio
const rateLimit = require('express-rate-limit');

const createLinkLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minuti
  max: 10, // 10 link per finestra
  message: 'Troppi link creati, riprova più tardi'
});
```

### Performance Optimization

**Database Indexing:**
```sql
-- Indici per performance
CREATE INDEX idx_links_user_id ON links(user_id);
CREATE INDEX idx_links_short_code ON links(short_code);
CREATE INDEX idx_clicks_link_id ON clicks(link_id);
CREATE INDEX idx_clicks_timestamp ON clicks(timestamp);
```

**Caching Strategy:**
```typescript
// Redis caching per link frequenti
interface CacheStrategy {
  linkCache: Map<string, Link>;     // Cache link attivi
  analyticsCache: Map<string, any>; // Cache calcoli analytics
  ttl: number;                      // Time to live
}
```

**Frontend Optimizations:**
```typescript
// Virtual scrolling per liste lunghe
import { FixedSizeList as List } from 'react-window';

// Lazy loading componenti
const Analytics = lazy(() => import('./pages/Analytics'));

// Memoization calcoli costosi
const expensiveCalculation = useMemo(() => {
  return computeAnalytics(links);
}, [links]);
```

### Features Avanzate

**PWA Support:**
```typescript
// Service Worker per offline
self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  }
});
```

**Real-time Analytics:**
```typescript
// WebSocket per aggiornamenti live
const socket = io('ws://localhost:3001');

socket.on('linkClicked', (data) => {
  updateAnalytics(data);
});
```

**Bulk Operations:**
```typescript
// Operazioni multiple
interface BulkOperations {
  deleteMultiple: (linkIds: string[]) => Promise<void>;
  updateMultiple: (updates: Partial<Link>[]) => Promise<void>;
  exportData: (format: 'csv' | 'json') => Promise<Blob>;
}
```

### Monitoring e Analytics

**Error Tracking:**
```typescript
// Sentry integration
import * as Sentry from '@sentry/react';

Sentry.captureException(error, {
  tags: {
    component: 'LinkCreation',
    userId: user.id
  }
});
```

**Performance Monitoring:**
```typescript
// Web Vitals tracking
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

getCLS(console.log);
getFID(console.log);
getFCP(console.log);
getLCP(console.log);
getTTFB(console.log);
```

---

## CONCLUSIONI

**LinkShort** rappresenta
