# Esxi

## Reload VM after UUID change

```bash
vim-cmd vmsvc/getallvms
vim-cmd vmsvc/reload $vmID
```
