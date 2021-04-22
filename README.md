# course-signup

Projeto Conceito

Este repo contém, nesta fase, apenas a estrutura conceitual e ainda não executa nenhuma ação ao ser executado. Ele está sendo utilizado como base para um projeto pessoal de cursos que estou construindo com API em C# Core e front em ReactJS ( no momento, fora do escopo). 

O objetivo principal é efetuar o registro de alunos em cursos utilizando o modelo de mensageria. Tanto o processamento das mensagens quanto as notificações são processados pelo conceito de CQRS.

Para este projeto está sendo considerado o uso do Kinesis para mensageria, CQRS para o disparos dos processos assincronos, e o fluxo de notificação via "Fire e Forget" onde não está sendo analizados o retorno do servidor SMTP (envio, validade do email, etc)

A quantidade de cursos, alunos e a relação dos cursos estão armazenados na tabela cfg-cursos do BD. Como a inscrição é feita de forma paralela via Threads o serviço de Sign-in (fluxo de inscrição) controla a quantidade de alunos via tabela de estatisticas.

Sempre que um aluno é inscrito, um evento é gerado (Services ORM) de forma a gravar um totalizador de registros. Este totalizador contempla:
- Quantidade de alunos do curso
- Metricas por curso (idade minima, idade máxima, média de idades)
- metricas do campus (idade minima, idade máxima, média de idades)

O presente modelo não contempla demais regras como idade minima, pagamento de taxas, entrega de documentos, email válido entre outras.

A leitura da tabela de estatisticas seria um serviço Singleton de forma a garantir a não colizão dos dados (inscrever mais alunos que a turma permite). Outra forma é deixar o BD gerenciar a qtd de inscritos (tipo SQL Server via Trigguer/Procedure). Ou usar uma logica FIFO para gerencia a assertividade de alunos na turma. Este algoritmo está fora do escopo no momento.

# EndPoins
## Registro
![Alt text](/course-signup-api/img/endpoints.JPG?raw=true "endpoints")
O EndPoint (EP) de registro efetua o agendamento da solicitação do aluno em um stream kinesis (mensageria). Neste conceito utilizando o elasticsearch para gerenciar a carga cfe o número de requisições aumente de forma exponencial.

A API tem como entradas:
- Nome
- Data de nascimento
- ID do Curso
- Email

Após ser incluido na fila é enviado um comando para que o serviço de notificação envie um email ao aluno informando sobre os proximos passos. Estes dados tambem pode ser exibidos no front (fora do escopo atual).

## Consulta
![Alt text](/course-signup-api/img/consultas.JPG?raw=true "consultas")

O EP de consulta retorna as metricas de alunos  (idade minima, idade máxima, média de idades) por cuso e campus. Serão implementados no futuro outros endpoints como:
- Relação de alunos por curso
- Quantidade de alunos inscritos
- Data da inscrição
- Etc

Este retorno pode ser feito por "Dynamic Querys" (DQ), GraphQL ou DTO's. Nesta versão será implementado o modelo de DQ.

# Processos

## Processamento
![Alt text](/course-signup-api/img/processamento-filas.JPG?raw=true "processamento")
O processamento é iniciado pelo evento "Processar Fila" onde o mesmo efetua:
- Inscrição do aluno na turma
- Geração das estatisticas
- Disparo do comando "Registrar" aluno

O correto deste evento é ser gerado por um scheduler, para garantir que todos os registros da fila seja processados, ou por um serviço independente (robo/loop) que fica monitorando a existencia de registros na fila.

## Inscrição
![Alt text](/course-signup-api/img/inscreicao.JPG?raw=true "inscricao")
A inscrição inicia pelo evento "Registro" onde é recuperado do kinessis os alunos. Este serviço verifica a qtd de alunos na turma, via tabela de estatisticas e gera o comando "Registrar". Se a turma estiver "cheia" os alunos entram no expurgo (não detalhado no fluxo) sendo notificados da "Turma Cheia". Este conceito é interessante se implementarmos outro midleware que valide regras adcionais "não online" de forma que no momento não se possível garantir que a turma foi fechada ou não. Este midleware esta fora do escopo.

## Notificação
![Alt text](/course-signup-api/img/notificacao.JPG?raw=true "notificacao")
A notificação "inicia" pelo evento "notificação" sendo responsavel pelo envio de emails aos alunos. de forma paralelizada (thread). São enviados 3 tipos de email por este processo:
- Notificação do agendamento
- Aviso de inscrição efetuada com sucesso
- Aviso de inscrição declinada por turma cheia

#Este repo é basicamente para POC's e testes em geral. Se tiver interesse em ver meus projetos reais envie um e-mail para webmaster@asgi.com.br explicando o motivo do acesso.
