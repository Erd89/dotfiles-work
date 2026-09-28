# Karabiner configuration (no stow)

Configurazione di [Karabiner-Elements](https://karabiner-elements.pqrs.org/) gestita
manualmente (non via `stow`).

Il file attivo è `~/.config/karabiner/karabiner.json`; la copia di riferimento in questo
repo è `karabiner.json` (e viene ricaricata da Karabiner da sola appena cambia).

## Regole

| Combinazione | Azione |
|---|---|
| `Cmd` + `F1` | spazio a sinistra (`/opt/homebrew/bin/noswoosh left`) |
| `Cmd` + `F2` | spazio a destra (`/opt/homebrew/bin/noswoosh right`) |
| `Alt` + `G`  | apre Ghostty |
| `Alt` + `F`  | apre Finder |

Nota: nel profilo `simple_modifications` c'è uno **swap Cmd ⇄ Option**, quindi le regole
scritte con `left_command` corrispondono al tasto fisico che fa da Command.

## ⚠️ Permessi Accessibility (il problema ricorrente)

`noswoosh left|right` non fa altro che **iniettare eventi sintetici** (la gesture del
Dock). macOS li blocca **in silenzio** se il processo che li invia non ha il permesso
**Accessibility** in *Impostazioni di Sistema → Privacy e Sicurezza → Accessibilità*.

Il punto che frega: macOS **non** valuta il permesso sul binario invocato, ma sul
**"responsible process"** che lo lancia (`noswoosh` gira come figlio, quindi conta il
padre). Risultato: la stessa `karabiner.json` funziona su una macchina e non su un'altra,
a seconda di chi ha il permesso.

Sintomo: la combinazione non fa nulla (nessun errore), anche se `noswoosh.app` risulta
già spuntato in Accessibilità.

### Cosa deve avere Accessibility

| Processo | Serve perché |
|---|---|
| **Karabiner-Elements** (helper/console user server) | è il padre di `shell_command` → `/opt/homebrew/bin/noswoosh …` |
| **Ghostty** (o il terminale usato) | è il padre quando lanci `noswoosh` a mano dalla shell |
| **noswoosh.app** | serve comunque per lanciarlo via `open -a` e per il daemon (Ctrl+frecce) |

### Verifica rapida

Compila un probe usa-e-getta e lancialo **nel contesto in cui gira il comando**:

```sh
cat > /tmp/axcheck.swift <<'EOF'
import ApplicationServices
print(AXIsProcessTrusted() ? "true" : "false")
EOF
swiftc /tmp/axcheck.swift -o ~/bin/axcheck
~/bin/axcheck        # "true" = Accessibilità concessa, "false" = negata
```

Per verificare il contesto di Karabiner, aggiungi a una regola una `shell_command`
temporanea che scrive l'esito, es.:

```
/bin/echo "trust=$(/Users/nunzio/bin/axcheck)" >> /tmp/kb_test.log
```

## Alternativa senza permessi: Ctrl+← / Ctrl+→

Per evitare del tutto la dipendenza da TCC, nella regola si possono far emettere le
frecce con Ctrl e lasciare il lavoro al **daemon** di noswoosh (che è già fidato, perché
`noswoosh setup` ha disattivato le hotkey Ctrl+frecce di sistema):

```json
"to": [{ "key_code": "left_arrow",  "modifiers": ["left_control"] }]
"to": [{ "key_code": "right_arrow", "modifiers": ["left_control"] }]
```

Questa variante funziona anche senza dare Accessibility a Karabiner/Ghostty.

## Contesto

- Verificato il **2026-09-28** su macOS **27.0 (26A428)** con noswoosh **1.7.6**.
- Con i permessi a Karabiner/Ghostty il binario diretto dal `shell_command` va
  (`trust=true`, spazio che cambia).
- Senza quei permessi il binario diretto fallisce **0/5**, mentre
  `open -n -a /Applications/noswoosh.app --args left` andava **5/5**.

## File duplicato

`no stow/karabiner-configuration/.config/karabiner/karabiner.json` è una variante più
vecchia (senza le regole noswoosh). Il riferimento aggiornato è `karabiner.json`.
