# Xifrat asimètric: com funciona

## Introducció i objectius

El xifrat asimètric fa servir dues claus relacionades: el que xifra una, només ho pot desxifrar l'altra. Aquesta idea resol el gran problema del xifrat simètric (com compartir la clau) i fa possibles HTTPS, SSH, la signatura digital i els certificats.

En acabar aquest material, l'alumnat ha de ser capaç de:

1. Explicar la diferència entre xifrat simètric i asimètric i quan s'utilitza cadascun.
2. Descriure el paper de la clau pública i de la clau privada en la confidencialitat i en la signatura.
3. Calcular a mà un exemple de RSA amb nombres petits.
4. Explicar per què a la pràctica es fa servir xifrat híbrid i què aporta una PKI.
5. Generar claus, xifrar, signar i verificar amb OpenSSL i GPG.

## El problema: distribuir la clau

En el xifrat simètric (AES, ChaCha20) s'utilitza la mateixa clau per xifrar i per desxifrar. És molt ràpid, però l'emissor i el receptor han de compartir aquesta clau abans de comunicar-se, i fer-ho per un canal insegur és justament el que volem evitar.

A més, el nombre de claus creix molt de pressa: n persones que volen parlar en privat dos a dos necessiten n(n−1)/2 claus. Una classe de 30 alumnes necessitaria 435 claus diferents.

|  | Simètric | Asimètric |
| --- | --- | --- |
| Claus | Una de compartida | Un parell per persona (pública + privada) |
| Claus per a n usuaris | n(n−1)/2 | 2n |
| Velocitat | Molt ràpid | Centenars o milers de vegades més lent |
| Ús típic | Xifrar grans volums de dades | Intercanviar claus, signar, autenticar |
| Exemples | AES, ChaCha20, 3DES | RSA, ECC (ECDSA, Ed25519), Diffie-Hellman |

La solució pràctica combina tots dos: l'asimètric per acordar una clau i el simètric per xifrar les dades (secció de xifrat híbrid).

## El parell de claus

Cada usuari genera un parell de claus matemàticament lligades: una **clau pública**, que es pot publicar a tothom, i una **clau privada**, que no surt mai del seu equip. El que es xifra amb una clau només es pot desxifrar amb l'altra del mateix parell.

- **Clau pública**: com un cadenat obert que reparteixes a tothom. Qualsevol el pot tancar, però només tu tens la clau que l'obre.
- **Clau privada**: la clau del cadenat. Si es perd o es filtra, tota la seguretat cau.

### Per què no es pot calcular la privada a partir de la pública?

Es basa en **funcions unidireccionals amb porta del darrere** (trapdoor): operacions fàcils en un sentit i pràcticament impossibles de desfer sense una dada secreta.

- **RSA**: multiplicar dos primers grans és immediat; factoritzar el producte (un nombre de 617 xifres per a RSA-2048) no és viable amb els ordinadors actuals.
- **Diffie-Hellman i ECC**: calcular una potència (o un múltiple d'un punt d'una corba) és fàcil; trobar l'exponent a partir del resultat (el *logaritme discret*) és molt difícil.

La seguretat és computacional, no absoluta: depèn de la mida de la clau i de la potència de càlcul disponible. Per això les longituds recomanades augmenten amb el temps, i els ordinadors quàntics són una amenaça a llarg termini (criptografia postquàntica).

## Confidencialitat: xifrar per a algú

Per enviar un missatge que només pugui llegir la Bob, l'Alice el xifra amb la **clau pública de la Bob**; només la **clau privada de la Bob** el pot desxifrar.

1. La Bob genera el seu parell de claus i publica la clau pública.
2. L'Alice obté la clau pública de la Bob.
3. L'Alice xifra el missatge amb aquesta clau pública i envia el text xifrat.
4. Un atacant que intercepti el missatge no pot fer-hi res: no té la clau privada.
5. La Bob desxifra amb la seva clau privada.

Regla per recordar: **per confidencialitat, es xifra amb la clau pública del destinatari.** Fixeu-vos que això no demostra qui ha enviat el missatge: qualsevol pot fer servir la clau pública de la Bob.

## Signatura digital: demostrar qui ho ha enviat

La signatura fa servir les claus a l'inrevés: l'Alice signa amb la seva **clau privada** i qualsevol pot verificar-ho amb la seva **clau pública**. Si la verificació funciona, només pot haver estat l'Alice.

No se signa tot el document, sinó el seu **resum (hash)**, que és curt i de mida fixa:

1. L'Alice calcula el hash del document (per exemple, SHA-256).
2. L'Alice xifra aquest hash amb la seva clau privada: això és la signatura.
3. Envia el document en clar + la signatura.
4. La Bob calcula ell mateix el hash del document rebut.
5. La Bob desxifra la signatura amb la clau pública de l'Alice i obté el hash original.
6. Si els dos hashes coincideixen, la signatura és vàlida.

| Propietat | Què garanteix | Per què |
| --- | --- | --- |
| Autenticitat | El missatge ve de l'Alice | Només ella té la clau privada |
| Integritat | El missatge no s'ha modificat | Un sol bit canviat dona un hash diferent |
| No repudi | L'Alice no pot negar que l'ha signat | La signatura no es pot generar sense la seva clau privada |

La signatura **no** dona confidencialitat: el document viatja en clar. Si calen les dues coses, es signa amb la privada de l'emissor i es xifra amb la pública del receptor.

&#91;embedded content: xifrat vs signatura · quina clau s'usa a cada pas\]

En el xifrat, la clau privada actua al final (qui rep); en la signatura, al principi (qui envia).

## Algorismes principals

### RSA (1977)

RSA és l'algorisme asimètric clàssic: serveix per xifrar i per signar, i la seva seguretat depèn de la dificultat de factoritzar nombres grans.

**Generació de claus**

1. Triar dos nombres primers p i q.
2. Calcular n = p × q (el mòdul, és públic).
3. Calcular φ(n) = (p − 1)(q − 1).
4. Triar e, coprimer amb φ(n) (sol ser 65537).
5. Calcular d, l'invers de e mòdul φ(n): e × d ≡ 1 (mod φ(n)).

Clau pública = (e, n). Clau privada = (d, n).

```latex
C = M^{e} \bmod n \qquad\qquad M = C^{d} \bmod n
```

**Exemple amb nombres petits** (comprovat amb Python)

| Pas | Càlcul | Resultat |
| --- | --- | --- |
| Primers | p = 5, q = 11 |  |
| Mòdul | n = 5 × 11 | 55 |
| φ(n) | (5 − 1)(11 − 1) | 40 |
| Exponent públic | e = 3 (mcd(3, 40) = 1) | 3 |
| Exponent privat | 3 × d ≡ 1 (mod 40) → 3 × 27 = 81 = 2 × 40 + 1 | d = 27 |
| Xifrar M = 7 | 7³ mod 55 = 343 mod 55 | C = 13 |
| Desxifrar | 13²⁷ mod 55 | M = 7 |
| Signar M = 7 (amb d) | 7²⁷ mod 55 | S = 28 |
| Verificar (amb e) | 28³ mod 55 | 7 = M, vàlida |

Amb Python es pot comprovar a classe: `pow(7, 3, 55)` retorna 13 i `pow(13, 27, 55)` retorna 7. Per calcular d: `pow(3, -1, 40)` retorna 27.

A la realitat, p i q tenen centenars de xifres i RSA no s'aplica directament al missatge, sinó amb un farciment (OAEP per xifrar, PSS per signar).

### Diffie-Hellman (1976)

No xifra ni signa: permet que dues parts **acordin una clau secreta compartida** per un canal públic, sense enviar-la mai.

|  | Alice | Públic (tothom ho veu) | Bob |
| --- | --- | --- | --- |
| Paràmetres |  | p = 23, g = 5 |  |
| Secret | a = 6 |  | b = 15 |
| Envia | A = 5⁶ mod 23 = 8 | A = 8, B = 19 | B = 5¹⁵ mod 23 = 19 |
| Calcula | 19⁶ mod 23 = **2** |  | 8¹⁵ mod 23 = **2** |

Tots dos obtenen la clau 2 sense haver-la transmès. Un atacant veu 23, 5, 8 i 19, però hauria de resoldre el logaritme discret per trobar a o b. Sense autenticació, DH és vulnerable a un atac d'home al mig (MITM); per això es combina amb certificats.

### Criptografia de corba el·líptica (ECC)

ECC ofereix la mateixa seguretat que RSA amb claus molt més curtes, i per això és més ràpida i és el que fan servir avui TLS 1.3 i SSH per defecte.

| Seguretat aproximada | RSA | ECC |
| --- | --- | --- |
| 128 bits | 3072 bits | 256 bits |
| 192 bits | 7680 bits | 384 bits |
| 256 bits | 15360 bits | 521 bits |

Noms habituals: **ECDH / X25519** (acord de claus), **ECDSA** i **Ed25519** (signatura).

## Xifrat híbrid i PKI

### Xifrat híbrid

Com que l'asimètric és lent, a la pràctica s'utilitza només per protegir una **clau de sessió** simètrica, i les dades es xifren amb AES o ChaCha20. Així s'aprofita el millor de cada sistema.

1. L'emissor genera una clau de sessió aleatòria (per exemple, AES-256).
2. Xifra les dades amb aquesta clau de sessió.
3. Xifra la clau de sessió amb la clau pública del receptor (o l'acorda amb Diffie-Hellman).
4. El receptor recupera la clau de sessió amb la seva privada i desxifra les dades.

Això és el que fan PGP/GPG, S/MIME, TLS i SSH.

### El problema de confiança: de qui és aquesta clau pública?

Si un atacant aconsegueix fer-nos creure que la seva clau pública és la del banc, pot llegir-ho tot (atac MITM). Cal una manera de lligar una clau pública amb una identitat.

- **Certificat digital (X.509)**: document que conté la clau pública, el nom del titular (p. ex. el domini), les dates de validesa i la **signatura d'una autoritat de certificació**.
- **Autoritat de certificació (CA)**: entitat de confiança que verifica la identitat i signa certificats (p. ex. Let's Encrypt, o a Catalunya el Consorci AOC per a idCAT).
- **Cadena de confiança**: CA arrel (preinstal·lada al sistema o al navegador) → CA intermèdia → certificat del servidor. El client verifica cada signatura fins arribar a una arrel en què confia.
- **Revocació**: un certificat compromès es pot invalidar abans que caduqui (CRL, OCSP).
- **Alternativa sense CA**: la *xarxa de confiança* de PGP, on els usuaris signen les claus dels altres; o el *fingerprint* que SSH demana acceptar la primera vegada.

### Tot junt: una connexió HTTPS (TLS 1.3, simplificat)

1. El client i el servidor fan un intercanvi Diffie-Hellman efímer (ECDHE) per acordar una clau.
2. El servidor envia el seu certificat i signa l'intercanvi amb la seva clau privada.
3. El client verifica el certificat amb la cadena de CA i comprova la signatura: el servidor és qui diu ser.
4. A partir de la clau acordada, tots dos deriven claus simètriques i les dades viatgen xifrades amb AES-GCM o ChaCha20-Poly1305.

Asimètric per autenticar i acordar claus, simètric per a les dades: el xifrat híbrid en acció.

