## 📖 Índice  
1. [Introdução](#introdução)  
2. [1. Conceitos Básicos](#1-conceitos-básicos)  
3. [2. Propriedades de um `Path`](#2-propriedades-de-um-path)  
4. [3. Listagem e Globbing](#3-listagem-e-globbing)  
5. [4. Operações no Sistema de Arquivos](#4-operações-no-sistema-de-arquivos)  
6. [5. Leitura e Escrita de Conteúdo](#5-leitura-e-escrita-de-conteúdo)  
7. [6. Métodos Avançados](#6-métodos-avançados)  
8. [7. Uso em Pipelines ETL / Data Engineering](#7-uso-em-pipelines-etl--data-engineering)  
9. [📚 Tabela de Métodos `pathlib`](#tabela-de-métodos-pathlib)  
10. [🚀 Exemplos Aprofundados](#exemplos-aprofundados)  

---

## 📚 Tabela de Métodos `pathlib`

| Método                                      | Descrição                                             |
|---------------------------------------------|-------------------------------------------------------|
| [Path.cwd()](#pathcwd)                      | Retorna o diretório de trabalho atual                 |
| [Path.home()](#pathhome)                    | Retorna o diretório home do usuário                   |
| [Path.resolve()](#pathresolve)              | Converte um path relativo em absoluto                 |
| [Path.exists()](#pathexists)                | Verifica se o arquivo/pasta existe                    |
| [Path.is_file()](#pathis_file)              | Verifica se é um arquivo                              |
| [Path.is_dir()](#pathis_dir)                | Verifica se é um diretório                            |
| [Path.iterdir()](#pathiterdir)              | Lista o conteúdo de um diretório                      |
| [Path.glob()](#pathglob)                    | Busca arquivos com padrão (não recursivo)             |
| [Path.rglob()](#pathrglob)                  | Busca arquivos com padrão (recursivo)                 |
| [Path.mkdir()](#pathmkdir)                  | Cria um diretório                                     |
| [Path.unlink()](#pathunlink)                | Remove um arquivo                                     |
| [Path.rename()](#pathrename)                | Renomeia ou move                                      |
| [Path.replace()](#pathreplace)              | Substitui e move, sobrescrevendo                       |
| [Path.symlink_to()](#pathsymlink_to)        | Cria um link simbólico                                |
| [Path.write_text()](#pathwrite_text)        | Escreve texto em um arquivo                           |
| [Path.read_text()](#pathread_text)          | Lê texto de um arquivo                                |
| [Path.write_bytes()](#pathwrite_bytes)      | Escreve bytes em um arquivo                           |
| [Path.read_bytes()](#pathread_bytes)        | Lê bytes de um arquivo                                |
| [Path.open()](#pathopen)                    | Abre o arquivo com controle manual                    |
| [Path.with_name()](#pathwith_name)          | Troca o nome do arquivo mantendo o diretório          |
| [Path.with_suffix()](#pathwith_suffix)      | Troca a extensão do arquivo                           |
| [Path.relative_to()](#pathrelative_to)      | Calcula o caminho relativo a outro                    |
| [Path.chmod()](#pathchmod)                  | Ajusta permissões numéricas                           |
| [Path.stat()](#pathstat)                    | Retorna informações detalhadas do arquivo ou pasta    |
| [Path.match()](#pathmatch)                  | Testa correspondência de padrão                       |

---

## Introdução

O módulo `pathlib` (Python 3.4+) fornece uma API orientada a objetos para caminhos de arquivo. Ele substitui (e simplifica) funções de `os.path`, `glob` e `os`.

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

| Operação        | Método                                 | Exemplo                               |
| --------------- | -------------------------------------- | ------------------------------------- |
| Criar pasta     | `p.mkdir()`                            | `Path('output').mkdir(exist_ok=True)` |
| Criar subpastas | `p.mkdir(parents=True, exist_ok=True)` | Cria hierarquia completa              |
| Remover arquivo | `p.unlink()`                           | `Path('a.txt').unlink()`              |
| Renomear/mover  | `p.rename(destino)`                    | `p.rename('novo_nome.txt')`           |
| Substituir      | `p.replace(destino)`                   | Move e sobrescreve se existir         |
| Criar symlink   | `p.symlink_to(target)`                 | Cria link simbólico                   |

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

## 🚀 Exemplos Aprofundados

### Path.cwd()

```python
from pathlib import Path
cwd = Path.cwd()
print(cwd)
```

---

### Path.home()

```python
from pathlib import Path
home = Path.home()
print(home)
```

---

### Path.resolve()

```python
from pathlib import Path
p = Path('data/../data/raw')
print(p.resolve())
```

---

### Path.exists()

```python
from pathlib import Path
print(Path('notas.txt').exists())
```

---

### Path.is\_file() / Path.is\_dir()

```python
p = Path('notas.txt')
print(p.is_file(), p.is_dir())
```

---

### Path.iterdir()

```python
from pathlib import Path
for entry in Path.cwd().iterdir():
    print(entry)
```

---

### Path.glob()

```python
from pathlib import Path
print(list(Path('src').glob('*.py')))
```

---

### Path.rglob()

```python
from pathlib import Path
print(list(Path('docs').rglob('*.md')))
```

---

### Path.mkdir()

```python
from pathlib import Path
Path('output/2025/report').mkdir(parents=True, exist_ok=True)
```

---

### Path.unlink()

```python
from pathlib import Path
Path('temp.txt').unlink()
```

---

### Path.rename()

```python
from pathlib import Path
Path('a.txt').rename('b.txt')
```

---

### Path.replace()

```python
from pathlib import Path
Path('b.txt').replace('output/b_final.txt')
```

---

### Path.symlink\_to()

```python
from pathlib import Path
Path('dest').symlink_to('origem')
```

---

### Path.write\_text() / Path.read\_text()

```python
from pathlib import Path
f = Path('exemplo.txt')
f.write_text("Olá!\n")
print(f.read_text())
```

---

### Path.write\_bytes() / Path.read\_bytes()

```python
from pathlib import Path
f = Path('data.bin')
f.write_bytes(b'\x00\xFF')
print(f.read_bytes())
```

---

### Path.open()

```python
from pathlib import Path
with Path('log.txt').open('a+') as fp:
    fp.write("Nova linha\n")
```

---

### Path.with\_name() / Path.with\_suffix()

```python
from pathlib import Path
p = Path('docs/report.txt')
print(p.with_name('sumario.txt'))
print(p.with_suffix('.md'))
```

---

### Path.relative\_to()

```python
from pathlib import Path
p = Path('/home/user/projeto/src/main.py')
print(p.relative_to('/home/user/projeto'))
```

---

### Path.chmod() / Path.stat()

```python
from pathlib import Path
p = Path('script.py')
p.chmod(0o744)
info = p.stat()
print(info.st_size, info.st_mode)
```

---

### Path.match()

```python
from pathlib import Path
p = Path('data/raw/info.txt')
print(p.match('data/*/*.txt'))
```

---

> **Dica Final**
> Sempre que lidar com caminhos — seja em automação, ETL, testes ou scripts diários — prefira `pathlib`. A API orientada a objetos é segura, legível e portátil entre sistemas operacionais.

