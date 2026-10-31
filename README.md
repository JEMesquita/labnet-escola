# LabNet Escola

Ambiente web para **laboratórios de redes e simulações de atividades**, criado para o curso Técnico em Redes de Computadores.

## O que a aplicação faz

| Recurso | Descrição |
|---|---|
| **Cadastro por e-mail** | Aluno se cadastra com nome, e-mail e senha e recebe um link de confirmação (válido por 48 h). Só entra depois de confirmar. |
| **Recuperação de senha** | Link por e-mail, válido por 1 h e de uso único. |
| **Turmas** | O professor cria turmas e compartilha um código. O aluno informa o código no cadastro ou depois, no perfil. |
| **Laboratório "Rede da Escola"** | Guia de montagem, arquivos `cfg-*.txt` para o Packet Tracer, topologia e 25 testes do roteiro, com registro de OK/FALHA e observação por aluno. |
| **Simulador de diagnóstico** | Terminal no navegador (ping, tracert, ipconfig, nslookup, show ...) em PCs, roteador e switches. Sorteia 8 tipos de falha; o aluno investiga e informa a causa. |
| **Painel do professor** | Progresso por aluno, detalhes dos testes e tentativas de diagnóstico, exportação CSV. |

O simulador **não é um emulador de IOS**: ele reproduz o comportamento da rede do laboratório (VLANs, ACLs, NAT, DHCP, DNS) e aceita apenas os comandos listados em `help`. Para praticar configuração de verdade, use o Packet Tracer (ver `docs/ambientes-virtuais.md`).

## Rodar localmente (teste)
Sem SMTP configurado, `DEV_SHOW_LINKS=1` mostra o link de confirmação na própria tela. Cadastre-se com o e-mail listado em `TEACHER_EMAILS` para ter o perfil de professor. Para promover alguém depois: `flask --app labnet promote`.

Testes automatizados:  (40 testes: cadastro, verificação, CSRF, papéis, laboratórios, simulador e API).

## Publicar (hospedagem)
Requisitos: um servidor Linux com Docker (VPS ou VM da escola), um domínio apontando para ele e uma conta SMTP para enviar os e-mails.

O Caddy obtém o certificado HTTPS automaticamente. Os dados ficam no volume `labnet-data` (arquivo SQLite). **Faça backup desse volume.**

Alternativas: qualquer plataforma que rode um contêiner ou um app Python/WSGI (comando: `gunicorn --workers 1 --threads 4 "labnet:create_app()"`), com um volume persistente para `DATABASE`.

## Segurança (o que já existe e o que falta)

Já existe: senhas com hash (Werkzeug), confirmação de e-mail, CSRF em todos os POST, cookies `HttpOnly`/`SameSite`, CSP restritiva, limite de tentativas de login, respostas de cadastro que não revelam se o e-mail já existe, consultas SQL parametrizadas, CSV protegido contra injeção de fórmulas e download restrito a uma lista de arquivos.

Recomendado antes de abrir para muitos usuários: limite de tentativas por IP em um proxy (o contador interno é em memória e por processo), backups automáticos, `ALLOWED_EMAIL_DOMAINS` se houver e-mail institucional e revisão do aviso de privacidade (LGPD) da escola, pois a aplicação guarda nome e e-mail de estudantes, inclusive menores.

## Estrutura

## Adicionar novos laboratórios

Edite `labnet/labs.py`: cada laboratório é um item de `LABS` com arquivos, testes e falhas. Coloque os arquivos em `labnet/lab_files/`. A topologia do simulador fica em `labnet/simulator.py` (hoje há um único cenário).
