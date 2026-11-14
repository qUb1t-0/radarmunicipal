# Radar Municipal

Painel local de prospecção de demandas municipais de mobiliário publicadas no PCA/PNCP. Uso individual, Windows, Python 3.12, Flask e SQLite, sem serviços pagos.

**Acesso:** abra `http://127.0.0.1:8765/` diretamente no Edge ou Chrome. As tarefas Windows iniciam painel e coleta; é necessário manter a sessão do usuário conectada.

```powershell
# Abrir/iniciar o painel
.\scripts\start-panel.ps1

# Coleta manual (respeita a trava de exclusão mútua)
& "$env:USERPROFILE\ProspecMunicipal\venv\Scripts\python.exe" -m prospeccao collect

# Parar execuções iniciadas pelo Agendador
Stop-ScheduledTask -TaskName 'ProspecMunicipal-Coleta'
Stop-ScheduledTask -TaskName 'ProspecMunicipal-Painel'
```

- [Manual de operação e recuperação](docs/operacao.md)
- [Validação e limites observados](docs/validacao.md)
- [Especificação aprovada](docs/superpowers/specs/2026-09-21-prospeccao-municipal-design.md)
- [Plano de implementação](docs/superpowers/plans/2026-09-21-prospeccao-municipal.md)
- [Validação técnica do PNCP](docs/research/pncp-validation.md)

O agendamento é de 2 horas. A carga nacional é progressiva e pode exigir muitos ciclos. Histórico não localizado permanece inconclusivo; o sistema não presume ausência de compra. Situação e anotações sobrevivem às atualizações da fonte.

Desde 24/09, o coletor separa novidades, carga inicial e revisitas, usa duas consultas simultâneas com limite compartilhado e espaça novas tentativas. A integração por data de atualização está implementada, mas os endpoints do PNCP devolveram respostas inválidas na validação real; os períodos ficam pendentes e as demais consultas continuam. Veja os detalhes e limites no manual de operação.

## Desenvolvimento

```powershell
& .\.venv\Scripts\python.exe -m pytest -q
& .\.venv\Scripts\python.exe -m pip check
& .\.venv\Scripts\python.exe -m compileall -q src
```

Dados reais ficam fora do repositório em `%USERPROFILE%\ProspecMunicipal`. Testes usam pastas temporárias e não consultam a internet. Regras em `config/keywords.json`; dependências fixadas em `requirements.lock.txt`.

## Valor mínimo das demandas — 23/09/2026

A pedido do usuário, somente demandas com valor total estimado **maior ou igual a R$ 500.000,00 por item do PCA** entram nas oportunidades e nos indicadores. Usa-se o valor publicado em `valorTotal`; não se somam itens, categorias ou planos de um órgão. Valores ausentes ou inferiores ficam fora, inclusive nos registros já coletados e ao incluir desativados. Registros anteriores e anotações são preservados; uma atualização de valor pode devolver a demanda à lista. O histórico de compras não recebe corte de valor, para preservar a prioridade correta. As consultas nacionais precisam ler os itens da fonte para verificar seu valor. O agendamento permanece de **2 horas**.

## Escopo atualizado — 23/09/2026

Esta lista substitui o vocabulário anterior:

| Combinação na mesma descrição | Categoria |
|---|---|
| mobiliário + escolar | Escolar |
| mobiliário + corporativo | Corporativo |
| cadeira + operacional | Operacional |
| mesa + escritório | Corporativo |
| móveis + planejados | Planejados e sob medida |
| mobiliário + sob medida | Planejados e sob medida |
| conjunto + aluno | Escolar |
| carteira + escolar | Escolar |

Os termos podem aparecer separados e em qualquer ordem. Aceitam-se singular/plural, maiúsculas/minúsculas, acentos e as variantes de gênero configuradas; “sob medida” também aceita hífen. A correspondência exige palavras inteiras. Os termos de duas descrições diferentes não são combinados. A configuração fica em `config/keywords.json` e aparece no painel em **Escopo da busca**.

Registros de demandas e compras históricas já armazenados são reclassificados ao aplicar a configuração. Anotações, situação comercial, evidências e contratos são preservados; categorias que saíram do escopo ficam desativadas. Os planos anteriormente consultados voltam à fila desde a primeira página, para recuperar itens que não eram armazenados pelo escopo anterior. O histórico mantém suas consultas concluídas. As revisitas são progressivas; a atualização não significa que todas as novas demandas já foram encontradas.

Mantidos mínimo de **R$ 500.000,00 por item do PCA**, prioridade por tempo desde a compra elegível, permanência de demanda nova com compra recente, coleta **a cada 2 horas** e contador.
