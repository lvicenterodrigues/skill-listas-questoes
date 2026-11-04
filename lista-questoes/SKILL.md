---
name: lista-questoes
description: Transforma imagens (prints) de questões de vestibulares/ENEM/concursos em uma lista de exercícios em LaTeX (.tex) compilada em PDF, transcrevendo tudo em texto e recortando apenas as figuras indispensáveis. Use quando o usuário fornecer imagens de questões e pedir uma lista de exercícios, prova ou apostila em LaTeX/PDF.
---

# Lista de questões a partir de imagens

Gera `lista.tex` + `figuras/` + `lista.pdf` a partir de imagens de questões.

## Fluxo

1. **Inventário**: liste as imagens do diretório (`ls`), incluindo `png/jpg/jpeg/webp`. Anote dimensões com PIL (`Image.open(f).size`).
2. **Leitura**: abra cada imagem com `read_file`. Identifique a origem (ex.: "ENEM 2020 -- Questão 150") pelo cabeçalho. Atenção: uma questão pode estar dividida em 2+ imagens (enunciado numa, alternativas em outra); una-as numa só questão. Se a origem não for identificável, pergunte ao usuário ou use "Questão N".
3. **Transcrição em texto** (regra principal: use texto sempre que possível):
   - Enunciado, citações/fontes (em `\footnotesize`), listas, itens e alternativas (A)–(E) em texto.
   - Matemática em modo math (`$...$`, `\dfrac`, `\text{ cm}`); unidades com `\text{}`; decimais com vírgula `{,}`; milhares com `\,`.
   - Tabelas em `tabular` com `booktabs`; mantenha os cabeçalhos curtos/quebrados em 2 linhas para evitar overfull.
   - Não resolva as questões nem invente dados; se algo estiver ilegível, marque com `% TODO` e avise o usuário.
4. **Figuras (só quando não dá para usar texto)**: gráficos, esquemas, mapas, fotos, desenhos geométricos. Recorte com PIL:
   ```python
   from PIL import Image
   Image.open(src).crop((x0,y0,x1,y1)).save('figuras/qN.png')
   ```
   - Recorte só a figura: sem fragmentos do texto do enunciado, sem marca d'água/cabeçalho, sem cortar rótulos/cotas. Deixe pequena margem.
   - Se a figura for simples (ex.: retângulo com diagonal), pode-se redesenhar em TikZ, mas o padrão é recortar.
   - Nome: `figuras/q<n>.png`. Inclua com `\includegraphics[width=0.5–0.7\textwidth]`.
5. **Montagem**: copie `templates/lista.tex` desta skill para o diretório de trabalho como `lista.tex`; preencha título/disciplina e use `\questao{origem}` + ambiente `alternativas` para cada questão. Ordene por número da questão (ou na ordem pedida).
6. **Compilação**: tente `pdflatex`, `xelatex` ou `tectonic` (`tectonic lista.tex`, que baixa pacotes). Rode duas vezes se houver referências. Corrija todos os `Overfull`/erros.
7. **Conferência visual (obrigatória)**: renderize `pdftoppm -r 60 -png lista.pdf /tmp/pv/p`, monte uma folha de contato com PIL e veja com `read_file`. Verifique:
   - todas as questões presentes e na ordem; numeração correta;
   - texto, números, unidades e alternativas idênticos às imagens;
   - figuras nítidas, sem sobras de texto nem cortes;
   - títulos não órfãos e questões não divididas entre páginas (use `\newpage` antes de questões longas; o template já usa `\Needspace`).
   Repita ajustes até ficar coerente.
8. **Entrega**: informe caminhos de `lista.tex`, `lista.pdf`, `figuras/`, o que foi recortado e qualquer dúvida de transcrição. Seja conciso.

## Observações
- Se nenhum compilador LaTeX existir, avise e entregue apenas o `.tex` (não alegue ter compilado).
- Preserve a língua original das questões (português por padrão; ajuste `babel` se necessário).
- Um exemplo completo está em `examples/` (lista com 8 questões do ENEM 2020).
