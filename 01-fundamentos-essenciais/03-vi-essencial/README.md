# 📘 Dicionário de Comandos do VI/VIM

Este guia prático lista os comandos mais úteis do editor `vi`/`vim`, organizados por categorias para facilitar seu uso durante os labs.

---

## 📝 Modo de Edição

| Comando | Ação                       |
| ------- | -------------------------- |
| `i`     | Inserir antes do cursor    |
| `I`     | Inserir no início da linha |
| `a`     | Inserir depois do cursor   |
| `A`     | Inserir no fim da linha    |
| `o`     | Nova linha abaixo          |
| `O`     | Nova linha acima           |
| `Esc`   | Sair do modo de inserção   |

---

## 💾 Salvar e Sair

| Comando | Ação                   |
| ------- | ---------------------- |
| `:w`    | Salvar                 |
| `:q`    | Sair                   |
| `:wq`   | Salvar e sair          |
| `:q!`   | Sair sem salvar        |
| `ZZ`    | Salvar e sair (atalho) |

---

## 🔍 Navegação

| Comando  | Ação                        |
| -------- | --------------------------- |
| `h`, `l` | Mover cursor ← →            |
| `j`, `k` | Mover cursor ↓ ↑            |
| `w`      | Avança uma palavra          |
| `b`      | Volta uma palavra           |
| `0`      | Início da linha             |
| `^`      | Primeiro caractere da linha |
| `$`      | Final da linha              |
| `G`      | Vai para a última linha     |
| `gg`     | Vai para a primeira linha   |
| `:n`     | Vai para a linha `n`        |

---

## ✂️ Copiar, Colar e Deletar

| Comando  | Ação              |
| -------- | ----------------- |
| `yy`     | Copiar linha      |
| `dd`     | Deletar linha     |
| `p`      | Colar abaixo      |
| `P`      | Colar acima       |
| `x`      | Deletar caractere |
| `u`      | Desfazer (undo)   |
| `Ctrl+r` | Refazer (redo)    |

---

## 🔄 Buscar e Substituir

| Comando            | Ação                                          |
| ------------------ | --------------------------------------------- |
| `/texto`           | Buscar "texto" para frente                    |
| `?texto`           | Buscar "texto" para trás                      |
| `n`, `N`           | Próxima / anterior ocorrência                 |
| `:%s/velho/novo/g` | Substituir "velho" por "novo" no arquivo todo |
| `:s/velho/novo/g`  | Substituir na linha atual                     |

---

Com esses comandos, você já consegue navegar, editar e salvar arquivos no `vi/vim` durante os labs com eficiência. Se quiser ir além, recomendo explorar os modos visuais (`v`, `V`, `Ctrl+v`) e macros.
