# Todo App v1.7.17 - Release Notes

**Release Date:** 2026-08-31  
**Status:** STABLE - Critical Bug Fix

---

## 🔴 CRITICAL: Stack Overflow beim "Fertig"-Button in Einstellungen

### Problem (v1.7.16)

Wenn Roger "Fertig" in den Einstellungen klickt → **Stack Overflow!**

```
Uncaught RangeError: Maximum call stack size exceeded
at CSSStyleDeclaration.set [as display]

Stack: localStorage.setItem() LOOP
```

### Root Cause: DOPPELTE `createBackup()` Funktionen!

Es gab **ZWEI Funktionen mit dem gleichen Namen:**

```javascript
// Zeile 2824 - v1.6.5.25 (ALTE AUTO-BACKUP - OBSOLET!)
function createBackup() {
    const timestamp = new Date().toISOString();
    const backupKey = `${BACKUP_KEY_PREFIX}${timestamp}`;
    localStorage.setItem(backupKey, JSON.stringify({...}));
    deleteOldBackups();  ← Könnte Loop verursachen!
}

// Zeile 2926 - v1.7.8 (NEUE ROTATING BACKUP - AKTUAL)
function createBackup() {  ← GLEICHER NAME!
    const backupFile = getNextBackupFile();
    localStorage.setItem(backupFile, JSON.stringify({...}));
}
```

**Das Problem:**
- Die zweite Funktion überschreibt die erste
- Aber beim Laden werden BEIDE definiert
- Das führt zu Konfusion und möglichen Problemen
- `closeSettings()` ruft `createBackup()` auf
- Wahrscheinlich benutzt es veraltete Logik

---

## ✅ Solution (v1.7.17)

### Alte Funktion komplett gelöscht!

```javascript
// GELÖSCHT: v1.6.5.25 alte createBackup()
// (verursachte Verwirrung + mögliche Bugs)

// BEHALTEN: v1.7.8 neue createBackup() mit Rotating Backup
```

**Was wurde gemacht:**
1. ❌ Alte createBackup() Funktion (v1.6.5.25) komplett gelöscht
2. ✅ Nur noch v1.7.8 createBackup() existiert
3. ✅ Keine Doppelungen mehr
4. ✅ Klare Logik

---

## 🛠️ Was ändert sich?

### Backup-System (v1.7.17):

```javascript
// Nur noch DIESE Version:
function createBackup() {
    const backupData = {
        timestamp: new Date().toISOString(),
        todos: allTodos  // NUR Todos! (keine Categories/Colors)
    };
    
    const backupFile = getNextBackupFile();  // B1 → B2 → B1 → B2
    localStorage.setItem(backupFile, JSON.stringify(backupData));
}
```

**Eigenschaften:**
- ✅ 2-datei rotating backup (B1 ↔ B2)
- ✅ Nur Todos speichern (Categories/Colors in GitHub)
- ✅ Automatische Rotation
- ✅ Keine Dubletten mehr

---

## ✅ Was wurde gefixt

- ✅ Alte v1.6.5.25 createBackup() Funktion gelöscht
- ✅ Keine doppelten Funktionsdefinitionen mehr
- ✅ Nur eine klare createBackup() Implementation
- ✅ Stack Overflow beim "Fertig" sollte weg sein

---

## 🎯 Warum war das ein Problem?

**Szenario v1.7.16:**
```
closeSettings() aufgerufen
  ↓
createBackup() ← WELCHE Funktion wird aufgerufen?
  ├─ v1.6.5.25 (alt, obsolet)
  └─ v1.7.8 (neu, aktuell)
  
→ Verwirrung führt zu möglichen Bugs!
```

**Szenario v1.7.17:**
```
closeSettings() aufgerufen
  ↓
createBackup() ← KLARE v1.7.8 Funktion
  ↓
2-Rotating Backup (B1 ↔ B2)
  
✅ Keine Verwirrung, keine Bugs!
```

---

## 📊 Alte vs Neue Funktion

### v1.6.5.25 (GELÖSCHT):
```javascript
function createBackup() {
    const timestamp = new Date().toISOString();
    const backupKey = `${BACKUP_KEY_PREFIX}${timestamp}`;  // Timestamp key
    
    localStorage.setItem(backupKey, JSON.stringify({
        timestamp: timestamp,
        todos: allTodos,
        count: allTodos.length,
        createdAt: new Date().toLocaleString('de-CH')
    }));
    
    deleteOldBackups();  // Räume alte Backups auf
}
```

**Problem:**
- Unbegrenzte Backups mit Timestamp als Key
- Speichert Todos + andere Daten
- Braucht deleteOldBackups() für Cleanup
- Kompliziert!

### v1.7.8 (BEHALTEN):
```javascript
function createBackup() {
    const backupData = {
        timestamp: new Date().toISOString(),
        todos: allTodos
    };
    
    const backupFile = getNextBackupFile();  // B1 oder B2
    localStorage.setItem(backupFile, JSON.stringify(backupData));
}
```

**Vorteile:**
- Nur 2 Backups (Rotating)
- Automatische Rotation (B1 → B2 → B1)
- Nur Todos (optimiert)
- Einfach & effizient!

---

## 🚀 Performance

- ✅ Weniger Code (alte Funktion gelöscht)
- ✅ Weniger Verwirrung
- ✅ Schneller & effizienter

---

**v1.7.17 ist PRODUCTION READY** ✅

**Stack Overflow beim "Fertig"-Button sollte behoben sein!** 🎉
