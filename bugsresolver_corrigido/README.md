# Central de Achados — Operação Resgate

Projeto Flutter corrigido para o desafio. O código foi revisado e os bugs documentados em `LAUDO.md` foram corrigidos, incluindo o fechamento correto do formulário de cadastro.

## Antes de executar: configuração obrigatória do Firebase

O ZIP original não contém `lib/firebase_options.dart`, porque esse arquivo precisa ser gerado para o projeto Firebase ao qual você tem acesso. Não é seguro inventar as credenciais nem substituir isso por um arquivo falso.

1. Abra a pasta do projeto no VS Code.
2. No terminal, execute `flutter pub get`.
3. Se a escola fornecer acesso a um projeto Firebase, configure o FlutterFire CLI e execute `flutterfire configure`; selecione o projeto correto e as plataformas necessárias. Isso gera `lib/firebase_options.dart`.
4. No console Firebase, ative **Authentication > Sign-in method > Google**, configure o domínio autorizado para Web e crie o banco Firestore.
5. Publique as regras do arquivo `firestore.rules` no Firestore (aba **Rules**).
6. Execute `flutter run -d chrome`.

Se a conta escolar não permitir acesso ao Firebase, peça ao professor as configurações/um projeto Firebase de teste ou autorização para usar outro projeto. Sem `firebase_options.dart` válido e sem Firebase configurado, não será possível testar o login e a sincronização entre contas. Não remova a autenticação nem desative as regras de segurança.

## Entrega
- Projeto Flutter corrigido.
- `LAUDO.md`, com sintomas, causas, correções e testes previstos.
- Captura de tela: faça login com duas contas Google em perfis/janelas diferentes do Chrome, registre uma ocorrência na conta A e mostre a mesma ocorrência na conta B.

Os testes de integração com Firebase precisam ser realizados depois da configuração; não afirmo que foram executados neste ambiente.
    