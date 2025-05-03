# Pathlib-docs
Fiz essa documentação de consulta para o PathLib do python

**Estruturando tutorial sobre pathlib**

O usuário quer aprender pathlib de forma simples e organizada. Vou criar um tutorial estruturado, começando com o básico e avançando até as funcionalidades mais complexas. O tutorial vai ser organizado em cinco módulos:

1. **Instalação e importação**
2. **Criando e manipulando caminhos**
3. **Operações comuns**
4. **Leitura e escrita de arquivos**
5. **Operações avançadas** (como globs, recursão, mover/copiar arquivos, etc.)


````markdown
# Guia Completo de `pathlib`: do Iniciante ao Avançado

Este guia mostra, em passos lógicos e exemplos simples, como usar o `pathlib` em Python para navegar e manipular arquivos e pastas.

---

## 📖 Índice  
1. [Introdução](#introdução)  
2. [Conceitos Básicos](#1-conceitos-básicos)  
3. [Propriedades de um `Path`](#2-propriedades-de-um-path)  
4. [Listagem e Globbing](#3-listagem-e-globbing)  
5. [Operações no Sistema de Arquivos](#4-operações-no-sistema-de-arquivos)  
6. [Leitura e Escrita de Conteúdo](#5-leitura-e-escrita-de-conteúdo)  
7. [Métodos Avançados](#6-métodos-avançados)  
8. [Uso em Pipelines ETL / Data Engineering](#7-uso-em-pipelines-etl--data-engineering)  

---

## Introdução

O módulo `pathlib` (**Python 3.4+**) fornece uma API orientada a objetos para caminhos de arquivo. Ele substitui (e simplifica) funções de `os.path`, `glob` e `os`.

```python
from pathlib import Path
````

---

## 1. Conceitos Básicos

| Ação                | Código                              | O que faz                              |
| ------------------- | ----------------------------------- | -------------------------------------- |
| CWD (current dir)   | `Path.cwd()`                        | Retorna o diretório de trabalho atual. |
| Home do usuário     | `Path.home()`                       | Retorna o diretório home (`~`).        |
| Criar path relativo | `p = Path('data') / 'raw' / '2025'` | Concatena pastas sem ligar `\` ou `/`. |
| Resolver absoluto   | `p.resolve()`                       | Converte p em caminho absoluto.        |

```python
from pathlib import Path

cwd  = Path.cwd()
home = Path.home()
p    = Path('data') / 'raw' / '2025'
print(cwd, home, p.resolve())
```

---

## 2. Propriedades de um `Path`

Com um objeto `Path` você pode:

```python
p = Path('/home/meuuser/projeto/script.py')

p.exists()        # True se existir
p.is_file()       # True se for arquivo
p.is_dir()        # True se for pasta
p.name            # 'script.py'
p.stem            # 'script'
p.suffix          # '.py'
p.parent          # PosixPath('/home/meuuser/projeto')
p.parents[1]      # '/home/meuuser'
p.anchor          # '/' (Unix) ou 'C:\' (Windows)
p.drive           # 'C:' (Windows) ou '' (Unix)
```

---

## 3. Listagem e Globbing

### 3.1. Listar conteúdos (não recursivo)

```python
for entry in Path.cwd().iterdir():
    print(entry.name)
```

### 3.2. Padrões com `glob`

```python
files_py  = list(Path('src').glob('*.py'))     # todos .py em src/
files_csv = list(Path('data').glob('*.csv'))   # todos .csv em data/
```

### 3.3. Recursivo com `rglob`

```python
all_txt = list(Path('docs').rglob('*.txt'))    # em todas subpastas
```

---

## 4. Operações no Sistema de Arquivos

| Operação        | Método                 | Exemplo                               |
| --------------- | ---------------------- | ------------------------------------- |
| Criar pasta     | `p.mkdir()`            | `Path('output').mkdir(exist_ok=True)` |
para criar subpastas em sequencia adicione `mkdir(parents=true, exist_ok=True)`
| Remover arquivo | `p.unlink()`           | `Path('a.txt').unlink()`              |
| Renomear/mover  | `p.rename(destino)`    | `p.rename('novo_nome.txt')`           |
| Substituir      | `p.replace(destino)`   | move e sobrescreve se existir         |
| Criar symlink   | `p.symlink_to(target)` | cria link simbólico                   |

```python
out = Path('output')
out.mkdir(exist_ok=True)
(Path('a.txt')).rename(out/'a_renomeado.txt')
```

---

## 5. Leitura e Escrita de Conteúdo

```python
f = Path('notas.txt')

# Escrever texto
f.write_text("Olá, pathlib!\n")

# Ler texto
conteudo = f.read_text()

# Escrever/ler bytes
(f.parent/'img.png').write_bytes(b'\x89PNG...')
data = f.read_bytes()

# Abrir como arquivo padrão
with f.open('a+') as fp:
    fp.write("outra linha\n")
```

---

## 6. Métodos Avançados

* **`with_name()` / `with_suffix()`**

  ```python
  p = Path('data/raw/2025/report.txt')
  p2 = p.with_name('sumario.txt')       # data/raw/2025/sumario.txt
  p3 = p.with_suffix('.md')             # data/raw/2025/report.md
  ```

* **`relative_to()`**

  ```python
  p = Path('/home/user/projeto/src/main.py')
  p.relative_to('/home/user/projeto')   # 'src/main.py'
  ```

* **`chmod()` / `stat()`**

  ```python
  p.chmod(0o644)
  info = p.stat()
  print(info.st_size, info.st_mode)
  ```

* **Comparação e `match()`**

  ```python
  if p.match('*/raw/*.txt'):
      print("Arquivo de raw!")
  ```

---

## 7. Uso em Pipelines ETL / Data Engineering

1. **Particionamento dinâmico**

   ```python
   from datetime import date
   base = Path('/mnt/data')
   d    = date.today()
   part = base / f"year={d.year}" / f"month={d.month:02d}"
   ```

2. **Iterar e filtrar**

   ```python
   for csv in part.glob('*.csv'):
       df = pd.read_csv(csv)
       # transformações…
       out = Path('/mnt/data/processed') / csv.name
       df.to_parquet(out.with_suffix('.parquet'))
   ```

3. **Integração remota (fsspec)**

   ```python
   import fsspec
   fs = fsspec.filesystem('s3')
   p  = Path('s3://bucket/data') / '2025-05-03'
   files = fs.glob(str(p/'*.parquet'))
   ```

---

### Dica Final

> **Sempre que lidar com caminhos** — seja em automação, ETL, testes ou scripts diários — prefira `pathlib`. A API orientada a objetos é segura, legível e portátil entre sistemas operacionais.

---

Agora você tem um fluxo de aprendizado:

1. **Básico** (construir e navegar)
2. **Propriedades** (checar, extrair nomes)
3. **Listagem** (iterdir/glob/rglob)
4. **Operações** (mkdir, unlink, rename)
5. **I/O** (read/write)
6. **Avançado** (manipulação fina, integração remota)

Bons caminhos com `pathlib`! 🚀

```
```
