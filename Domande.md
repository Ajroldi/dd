# DOMANDA 1
Ecco dove sono implementate le funzioni che hai chiesto:

Funzioni di Scadenza Link:
File: src/pages/Redirect.tsx

Righe 20-24: Controllo se il link è scaduto prima del redirect

if (link.expiresAt && isAfter(new Date(), new Date(link.expiresAt))) {
  // Link scaduto - redirect alla home
  window.location.href = '/';
  return;
}
File: src/components/LinkCard.tsx

Riga 13: Calcolo se il link è scaduto per la visualizzazione

const isExpired = link.expiresAt ? isAfter(new Date(), new Date(link.expiresAt)) : false;
Righe 65-69: Visualizzazione dello stato di scadenza nell'interfaccia
File: src/pages/Analytics.tsx

Righe 56-61: Filtro dei link attivi (non scaduti) per le statistiche

return filteredLinks.filter(link => {
  if (!link.expiresAt) return true;
  return new Date(link.expiresAt) > new Date();
}).length;
Funzioni di Creazione Link Breve:
File: src/utils/storage.ts

Righe 35-42: Generazione del codice breve casuale

export const generateShortCode = (): string => {
  const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let result = '';
  const length = Math.floor(Math.random() * 3) + 4; // Random length between 4-6 characters
  for (let i = 0; i < length; i++) {
    result += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return result;
};
Righe 44-47: Verifica disponibilità del codice
Righe 49-55: Validazione URL
File: src/components/CreateLinkForm.tsx

Righe 75-85: Logica principale di creazione del link

let shortCode: string;

if (customCode && customCode.trim()) {
  shortCode = customCode.trim();
} else {
  do {
    shortCode = generateShortCode();
  } while (!isShortCodeAvailable(shortCode));
}
Righe 87-99: Creazione dell'oggetto link e salvataggio
Queste sono le implementazioni principali delle funzionalità che hai chiesto.
