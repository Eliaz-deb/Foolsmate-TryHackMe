# 🎯 TryHackMe - Fools Mate Writeup

**Room:** [Fools Mate](https://tryhackme.com/room/foolsmate)  
**Difficulté:** Easy  
**Catégorie:** Web Exploitation / Client-Side Bypass  
**Date:** Septembre 2026

---

## 📋 Résumé

Fools Mate est un challenge web qui met en lumière une faille de sécurité classique : **la validation côté client**. Le site propose un jeu d'échecs où un mouvement gagnant (échec et mat en 1 coup) est bloqué par le navigateur, mais le serveur ne vérifie pas la légalité du mouvement. En interceptant et modifiant la requête HTTP, on peut forger le mouvement gagnant et obtenir le flag.

---

## 🔍 Étape 1 : Reconnaissance

### Scan Nmap

J'ai commencé par identifier les services actifs sur la machine cible :

```bash
nmap -sV -Pn -T4 <IP_MACHINE>
```

**Résultats du scan :**
- **Port 22/TCP** : OpenSSH 9.6p1 (aucune vulnérabilité exploitable connue)
- **Port 80/TCP** : Serveur web Node.js

➡️ **Analyse :** Le port SSH n'étant pas vulnérable, je me concentre sur le service web.

---

## 🌐 Étape 2 : Analyse du Site Web

En accédant à `http://<IP_MACHINE>:80`, je découvre un **jeu d'échecs interactif**.

**Observations :**
- La position de l'échiquier permet un échec et mat en 1 coup (de `a1` à `a8`)
- L'interface ne propose pas ce mouvement gagnant
- Seul des mouvements légaux "normaux" sont autorisés

**Hypothèse :** Le site utilise une validation côté client (JavaScript) pour empêcher certains mouvements, mais le serveur pourrait ne pas vérifier la légalité des coups.

---

## 🔎 Étape 3 : Analyse du Trafic Réseau

### Outil utilisé
Inspecteur d'éléments du navigateur (F12) → Onglet **Network**

### Méthodologie
1. Ouvrir les DevTools (touche F12)
2. Aller dans l'onglet **Network** (Réseau)
3. Effectuer un mouvement légal sur l'échiquier (ex: déplacer un pion de `a1` à `a2`)
4. Observer la requête HTTP générée

### Requête interceptée
```http
POST /move
Content-Type: application/json

{"move": "a1a2"}
```

➡️ **Constat :** Le site envoie le mouvement sous forme de chaîne de caractères (`a1a2` = de la case a1 à la case a2).

---

## 💥 Étape 4 : Exploitation - Bypass de la Validation Client

### Faille identifiée
Le serveur accepte n'importe quel mouvement **sans vérifier** s'il est légal selon les règles des échecs. C'est une **faille de validation côté client**.

### Attaque
Modifier la requête pour envoyer le mouvement gagnant (`a1` → `a8`).

**Requête forgée :**
```http
POST /move
Content-Type: application/json

{"move": "a1a8"}
```

### Résultat
Le serveur accepte le mouvement et retourne le flag ! 🎉

---

## 🚩 Flag

```
THM{cl13nt_s1d3_ch3ckm4t3}
```

---
