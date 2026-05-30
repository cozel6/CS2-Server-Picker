# CS2 Server Picker

Aplicatie desktop pentru Windows care blocheaza serverele Valve din regiunile
nedorite (de exemplu Rusia si Turcia) folosind Windows Firewall. Scopul este ca
matchmaking-ul din CS2 sa prefere serverele din Europa de Vest (Frankfurt), ca sa
joci cu o latenta mai buna si alaturi de jucatori predominant din EU West.

## Ce vrem sa facem

1. Afisam o lista de regiuni (RU, TR, EU West, etc.).
2. Bifam regiunile pe care vrem sa le blocam.
3. Apasam "Apply" si aplicatia adauga regulile in Windows Firewall.
4. Dam play in CS2, iar matchmaking-ul evita serverele blocate.
5. Apasam "Reset" si toate regulile adaugate de aplicatie sunt sterse.

## Cum functioneaza pe scurt

```
Tu (RO) -> CS2 cauta server
        -> serverele RU/TR sunt blocate in firewall
        -> CS2 gaseste doar servere Frankfurt / EU West
        -> jucatori predominant din EU West
```

Important: blocam servere, nu jucatori. Un jucator din RU se poate conecta
totusi pe un server din Frankfurt.

## Implementarea fiecarui fisier

Acesta este planul de implementare. Codul efectiv il scriem pas cu pas.

### main.py
Punctul de intrare al aplicatiei. Verifica daca programul ruleaza ca
Administrator (necesar pentru firewall), apoi porneste fereastra principala din
`gui/app.py`.

### core/firewall.py
Motorul aplicatiei. Comunica cu Windows Firewall prin `subprocess` si `netsh`.
Functii planificate:
- adaugare regula de blocare outbound catre un IP range
- stergere a unei reguli dupa nume
- listare a regulilor existente create de aplicatie
- reset complet (sterge toate regulile cu prefix `CS2SP_`)

Toate regulile create folosesc prefixul `CS2SP_` in nume, ca sa fie usor de
identificat si de sters fara a afecta alte reguli din firewall.

### core/regions.py
Incarca `data/regions.json` si ofera logica pentru regiuni si IP ranges.
Functii planificate:
- citire si validare a fisierului JSON
- returnarea listei de regiuni catre GUI
- maparea unei regiuni la IP range-urile ei, pentru `firewall.py`

### core/profiles.py
Gestioneaza profilurile salvate (combinatii de regiuni bifate).
Functii planificate:
- salvarea unui profil pe disc
- incarcarea unui profil existent
- listarea si stergerea profilurilor

### gui/app.py
Fereastra principala construita cu customtkinter. Aduce la un loc lista de
region cards, panoul de profiluri si butoanele "Apply" si "Reset".

### gui/region_card.py
Un card vizual pentru o singura regiune: checkbox, steag si numele regiunii.
Tine starea (bifat / nebifat) pentru fiecare regiune.

### gui/profile_panel.py
Panoul lateral pentru profiluri: salvare profil nou, alegere profil existent,
stergere profil.

### data/regions.json
Datele cu IP range-urile serverelor Valve, grupate pe regiuni. Se poate
actualiza ocazional, deoarece IP-urile Valve se pot schimba in timp.

### assets/flags/
Iconite cu steaguri (png) folosite de region cards.

## Limitari cunoscute

| Aspect | Realitate |
|---|---|
| Blocam servere, nu jucatori | Un RU poate aparea pe server Frankfurt |
| IP-urile Valve se pot schimba | `regions.json` necesita update ocazional |
| Necesita Administrator | Pentru a modifica firewall-ul |
| Timp matchmaking mai mare | Mai putine servere disponibile |
| VPN nu ajuta la ping | Adauga overhead, nu scade ms-urile |
