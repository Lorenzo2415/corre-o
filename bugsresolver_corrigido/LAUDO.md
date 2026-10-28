# LAUDO TÉCNICO — Operação Resgate: Detetives de Bugs

Projeto: Central de Achados (Flutter + Firebase Auth + Firestore)

## Bug 01
- **Arquivo:** pubspec.yaml / test/widget_test.dart
- **Sintoma:** `flutter pub get`/`flutter test` falham: o teste importa `package:bugsresolver/main.dart` e usa `MyApp`, mas o pacote se chamava `central_achados`.
- **Causa:** Nome do pacote no `pubspec.yaml` não batia com o projeto; o teste era o template de contador, que não existe no app.
- **Correção:** `name: bugsresolver`; teste de template removido.
- **Teste:** `flutter pub get` e `flutter analyze` sem erros de import.

## Bug 02
- **Arquivo:** lib/main.dart (`firebase_options.dart`)
- **Sintoma:** Não compila: `Target of URI doesn't exist: 'firebase_options.dart'`.
- **Causa:** O arquivo gerado pelo FlutterFire não vem no ZIP (o README avisa).
- **Correção:** Rodar `flutterfire configure` no projeto Firebase da escola para gerar `lib/firebase_options.dart` (nenhum código removido).
- **Teste:** `flutter run -d chrome` abre a tela de login.

## Bug 03
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Erro de compilação na linha do subtítulo (string com quebra de linha real).
- **Causa:** Aspas simples não aceitam quebra literal.
- **Correção:** Trocado a quebra por `\n` dentro da string.
- **Teste:** `flutter analyze` sem erro; subtítulo aparece em duas linhas.

## Bug 04
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Erro de compilação: `DetalhePage` exige `ocorrencia`.
- **Causa:** `DetalhePage()` chamado sem argumento (e o detalhe nunca saberia qual caso abrir).
- **Correção:** `DetalhePage(ocorrencia: o)`.
- **Teste:** Tocar em um item abre os dados daquele item.

## Bug 05
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Registrava objeto com título ou local vazio.
- **Causa:** Validação usava `&&`: só barrava se os dois estivessem vazios.
- **Correção:** Trocado por `||`.
- **Teste:** Tentar registrar só com título (ou só local) mostra a mensagem e não grava.

## Bug 06
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Switch “Somente não resolvidos” funcionava ao contrário.
- **Causa:** `onChanged` gravava `!value`.
- **Correção:** `somenteAbertos = value`.
- **Teste:** Ligar o switch mantém o estado ligado.

## Bug 07
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Com o filtro ligado, apareciam só os resolvidos.
- **Causa:** Filtro usava `o.resolvido` em vez de `!o.resolvido`.
- **Correção:** `where((o) => !o.resolvido)`.
- **Teste:** Com um caso resolvido e outro aberto, o filtro mostra só o aberto.

## Bug 08
- **Arquivo:** lib/models/ocorrencia.dart
- **Sintoma:** Todo caso aparecia “Em aberto” quando resolvido e vice-versa; botão do dono com texto trocado.
- **Causa:** `resolvido` era lido negado: `!(d['resolvido'] ...)`.
- **Correção:** `resolvido: d['resolvido'] as bool? ?? false`.
- **Teste:** Marcar como resolvido muda o rótulo para “Resolvido”.

## Bug 09
- **Arquivo:** lib/models/ocorrencia.dart
- **Sintoma:** Ninguém era reconhecido como dono; botão de status nunca aparecia.
- **Causa:** `autorId` lia o campo `autorNome`.
- **Correção:** `autorId: d['autorId']`.
- **Teste:** Na conta A (autora) o botão aparece; na B não.

## Bug 10
- **Arquivo:** lib/services/central_service.dart (registrar)
- **Sintoma:** Dono incorreto no documento (e-mail em vez de uid).
- **Causa:** `'autorId': u.email` – a checagem de dono compara com `uid`.
- **Correção:** `'autorId': u.uid`.
- **Teste:** No Firestore, o campo `autorId` do novo documento é o uid da conta que registrou. (Documentos antigos criados com e-mail precisam ser apagados/recriados.)

## Bug 11
- **Arquivo:** lib/services/central_service.dart (alterarStatus)
- **Sintoma:** Marcar como encontrado não mudava nada.
- **Causa:** Gravava o valor atual (`item.resolvido`) em vez do invertido.
- **Correção:** `update({'resolvido': !item.resolvido})`.
- **Teste:** Clicar alterna entre resolvido e em aberto, ao vivo nas duas contas.

## Bug 12
- **Arquivo:** lib/services/central_service.dart (enviarPista)
- **Sintoma:** Pistas apareciam em todos os casos ou em nenhum (iam para `ocorrencias/pistas/pistas`).
- **Causa:** Usava o id fixo `'pistas'` em vez de `ocorrenciaId`.
- **Correção:** `.doc(ocorrenciaId).collection('pistas')`.
- **Teste:** Pista enviada na conta B aparece só na ocorrência certa, na conta A também.

## Bug 13
- **Arquivo:** lib/services/central_service.dart (acompanhar)
- **Sintoma:** Casos mais novos ficavam no fim da lista.
- **Causa:** `orderBy('criadoEm', descending: false)`.
- **Correção:** `descending: true`.
- **Teste:** O último registrado aparece no topo.

## Bug 14
- **Arquivo:** lib/pages/detalhe_page.dart (enviar)
- **Sintoma:** Digitar uma pista e enviar não fazia nada; campo vazio “enviava”.
- **Causa:** Condição invertida: `if (pista.text.trim().isNotEmpty) return;`.
- **Correção:** `isEmpty`.
- **Teste:** Texto preenchido envia; vazio é ignorado.

## Bug 15
- **Arquivo:** lib/pages/detalhe_page.dart (enviar)
- **Sintoma:** A pista era gravada com mensagem vazia.
- **Causa:** Chamava `enviarPista(id, '')` em vez de passar o texto.
- **Correção:** `enviarPista(id, pista.text)`.
- **Teste:** A pista aparece com o texto digitado e o nome do autor.

## Segurança (Firestore)
Autenticação e regras foram mantidas. Foi adicionado `firestore.rules`: só logados leem; só o dono (`autorId == uid`) altera `resolvido`; pistas só podem ser criadas com o próprio uid. Além da checagem no app (`alterarStatus`), isso barra alteração direta no banco.

## Teste das duas contas
Duas janelas anônimas/perfis do Chrome, contas Google A e B: A registra → aparece em B ao vivo; B envia pista → aparece na ocorrência correta em A e B; B não vê o botão de status e A consegue alternar.

## Correção adicional identificada durante a revisão
- **Arquivo:** `lib/pages/central_page.dart` (`cadastrar`)
- **Sintoma:** após cadastrar uma ocorrência, o app podia fechar a tela principal em vez de fechar apenas a janela de cadastro.
- **Causa:** o `Navigator.pop()` usava o contexto da página principal, não o contexto do diálogo.
- **Correção:** a função recebe `dialogContext` e fecha o diálogo usando esse contexto.
- **Teste:** revisão do fluxo de navegação; confirme manualmente após configurar o Firebase.
