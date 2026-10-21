# Busca de Séries (Flask + Ajax)

Busca ao vivo de séries: a cada letra digitada, a página pergunta ao servidor e atualiza a lista **sem recarregar**.

Projeto de estudo baseado no exercício de Ajax com JSON do CS50 (Harvard), montado e depurado passo a passo.

## O que ele faz

1. Você abre a página e vê um campo de texto.
2. Digita `office`.
3. Aparece a lista: `The Office` e `Office Space`.

Se o campo estiver vazio, a lista some.

## Tecnologias

| Tecnologia | Para que serve aqui |
|---|---|
| **Python + Flask** | servidor web que recebe o pedido e responde |
| **SQLite (cs50.SQL)** | banco de dados com a tabela `shows` |
| **JavaScript (fetch + JSON)** | pede os dados ao servidor e monta a lista |
| **HTML** | o campo de texto e a lista `<ul>` |

## Estrutura de arquivos

```
Projeto01/
├── app.py              # servidor Flask (rotas "/" e "/search")
├── templates/
│   └── index.html      # página com o campo, a lista e o JavaScript
├── requirements.txt    # dependências do projeto
├── .gitignore          # o que não vai para o GitHub (venv, __pycache__, *.db)
└── README.md
```

O `shows.db` e a pasta `venv/` não ficam no Git. O banco é criado pelo comando da próxima seção.

## Como rodar

```bash
# 1. criar e ativar o ambiente virtual
python3 -m venv venv
source venv/bin/activate

# 2. instalar as dependências
pip install -r requirements.txt

# 3. criar o banco de teste com 5 séries
sqlite3 shows.db "CREATE TABLE shows (id INTEGER PRIMARY KEY, title TEXT); INSERT INTO shows (title) VALUES ('The Office'), ('Office Space'), ('Breaking Bad'), ('Friends'), ('The Crown');"

# 4. iniciar o servidor
flask run
```

Abra http://127.0.0.1:5000/ no navegador.

Dica: toda vez que abrir um terminal novo, rode `source venv/bin/activate` antes do `flask run`. O prompt mostra `(venv)` quando está ativo.

## Como funciona (passo a passo)

```
você digita → JavaScript → fetch → Flask → SQLite → JSON → JavaScript → lista na tela
```

**1. Você digita uma letra.** O navegador dispara o evento `input`, e o JavaScript captura:

```javascript
input.addEventListener('input', async function() { ... });
```

**2. O JavaScript chama o servidor.** O `fetch` pede a rota `/search` e manda o texto digitado no parâmetro `q`:

```javascript
let response = await fetch('/search?q=' + input.value);
```

**3. O Flask lê o `q` e consulta o banco.** O `?` no SQL é um parâmetro, e protege contra *SQL injection*:

```python
q = request.args.get("q")
shows = db.execute("SELECT * FROM shows WHERE title LIKE ? LIMIT 50", "%" + q + "%")
```

O `%` dos dois lados significa "o texto pode estar em qualquer parte do título".

**4. O Flask devolve JSON**, que é dado puro, e não uma página:

```python
return jsonify(shows)
# [{"id": 1, "title": "The Office"}, {"id": 2, "title": "Office Space"}]
```

**5. O JavaScript monta a lista.** Converte o JSON, cria um `<li>` para cada série e coloca tudo no `<ul>`:

```javascript
let shows = await response.json();
// for ... html += '<li>' + title + '</li>';
document.querySelector('ul').innerHTML = html;
```

Isso se chama **Ajax**: a página pede dados em segundo plano e atualiza só uma parte dela.

## Conceitos usados

- **Rota web:** `@app.route("/search")` liga uma URL a uma função Python.
- **Parâmetro de URL:** `request.args.get("q")` lê o que vem depois do `?`.
- **API JSON:** `jsonify(shows)` devolve dados, não uma página.
- **SQL com parâmetro:** o `?` no `LIKE ?` impede SQL injection. Nunca monte SQL concatenando texto do usuário.
- **Ajax / fetch:** o JavaScript pede dados sem recarregar a página.
- **async/await:** o navegador espera a resposta do servidor sem travar.
- **DOM:** `innerHTML` atualiza só a lista.
- **XSS:** o `replace` tenta evitar a injeção de HTML, mas troca só a primeira ocorrência. A proteção completa é usar `textContent` (ver "Próximos passos").

## Erros que apareceram no caminho (e o que cada um ensinou)

| Erro | Causa | Solução |
|---|---|---|
| `SyntaxError: invalid syntax` na linha do SQL | faltava o `+` em `"%" + q + "%"` | o Python mostra a linha exata do erro |
| `ModuleNotFoundError: No module named 'cs50'` | o Flask rodava fora do ambiente virtual | `source venv/bin/activate` e `pip install` |
| `TemplateNotFound: index.html` | a pasta se chamava `templantes` | renomear para `templates` |
| `Internal Server Error` (500) na busca | o `shows.db` estava vazio (0 bytes) | criar a tabela `shows` com o `sqlite3` |
| `Address already in use` | outro `flask run` ainda rodava na porta 5000 | `pkill -f "flask run"` ou `Ctrl+C` na outra aba |

Lição principal: o navegador só mostra "500". O **motivo real** está no terminal do `flask run`.

## Como investigar o que acontece por baixo

1. Abra o painel de desenvolvedor do navegador (`Ctrl + Shift + I`) e vá na aba **Rede** (Network).
2. Digite `office` letra por letra, **sem vírgulas**.
3. Clique em cada pedido `search?q=...` e veja a URL, o status `200` e a resposta em JSON.

Resultado observado: **um pedido por tecla digitada**. Por isso existe a técnica de *debounce* (ver abaixo).

## Próximos passos

- [ ] Trocar `innerHTML` por `textContent` (corrige o XSS de verdade)
- [ ] Debounce: esperar ~300 ms depois da última letra antes do `fetch`
- [ ] Mostrar "nenhum resultado" quando a lista vier vazia
- [ ] Mais colunas (ano, gênero) e `ORDER BY`
- [ ] Tratar erro do servidor com `try/catch`
- [ ] Teste automatizado com `pytest`
- [ ] Deploy online

## O que aprendi

- Python, HTML e JavaScript se conectam por meio de uma API: o JavaScript chama uma rota do Flask, que consulta o banco e responde em JSON.
- Um erro 500 não diz nada sozinho. O terminal do servidor mostra a causa, e foi assim que achei a pasta com nome errado, o `cs50` faltando no ambiente virtual e o banco vazio.
- O navegador faz um pedido ao servidor a cada tecla digitada. Quero aprender a limitar isso com debounce.
- Fiz meu primeiro commit no Git e organizei o projeto com `requirements.txt` e `.gitignore`.