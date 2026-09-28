
# Dotfiles Repository

This repository contains the configuration files for various tools and applications I use. 
The files are managed using [GNU Stow](https://www.gnu.org/software/stow/), which simplifies the process of managing symlinks for the configurations.

## Notes / Troubleshooting

- **Karabiner `shell_command` non fa nulla** (es. `noswoosh left`): è quasi sempre un
  problema di permesso **Accessibility** attribuito al processo padre, non al binario.
  Dettagli, verifica e alternativa senza permessi:
  [`no stow/karabiner-configuration/README.md`](no%20stow/karabiner-configuration/README.md).
