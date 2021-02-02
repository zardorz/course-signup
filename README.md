# course-signup

Projeto Conceito
Este repo contém, nesta fase, apenas a estrutura conceitual e ainda não executa nenhuma ação ao ser executado. Ele está sendo utilizado como base para um projeto pessoal de cursos que estou construindo com API en c# Core e front em ReactJS ( no momento, fora do escopo). 

O objetivo principal é efetuar o registro de alunos em cursos utilizando o modelo de mensageria. Tanto o processamento das mensagens qtos as notificações são processaçdos pelo conceito de CQRS.

Para este poehto está sendo considerado o uso do Kinesis para mensageria, CQRS para o disparos dos processos assincronos, e o fluxo de notificação via Fire e Forget onde não está sendo analziado o retorno do servidor SMTP (envio, validade do email, etc)

A quantidade de cursos alunos e os cursos estão armazaenados na tabela cfg-cursos do BD e quando a turma estiver fechada (analitics_course) a incrição é rejeitada. Como a inscrição é feita de forma paralela via Threads é o serviço de Sign-in que controla a qtd de alunos. Este controle é  por um select na tabela de estatísticas do curso. Um adendo...O conceito autal considera uma thread por curso. Caso as threads seja independentes, ou seja paralelizados, o controle pode ficar na Service-Filas, que é sincrona, recuperando apenas a quantidade correta de alunos da turma do kinesis (por ordem de data de solicitação por exemplo). Os demais são deprezados (e notificados). 

Sempre que um aluno é inscrito, um evento é gerado (Services ORM) de forma a gravar um totalizador de registros. Este totalizador contempla:
- Quantidade de alunos do curso
- Metricas por curso (idade minima, idade máxima, média de idades)
- metricas do campus (idade minima, idade máxima, média de idades)

O presente modelo não contempla demais regras como idade minima, pagamento de taxas, entrega de documentos, email valido entre outras que existiriam em um projeto real.


# EndPoins
## Registro
![Alt text](/course-signup-api/img/endpoints.JPG?raw=true "endpoints")
O EndPoint (EP) de registro efetua o agendamento da solicitação do aluno em um stream kinesis (mensageria). Neste conceito utilizando o elasticseach para gerenciar a carga cfe o numero de requisições aumente de forma exponencial.

A API tem como entrada seguintes dados:
- Nome
- Data de nascimento
- ID do Curso
- email

Após ser incluido na fila é enviado um comando para que o serviço de notificação envie um email ao aluno informando sobre os proximos passos. Estes dados tambem pode ser exibidos no front (fora do escopo atual).

##Consulta
![Alt text](/course-signup-api/img/consultas.JPG?raw=true "consultas")

O EP de consulta retorna as metricas de alunos  (idade minima, idade máxima, média de idades) por cuso e campus. Serão implementados no futuro outros endpoints como:
- Relação de alunos por curso
- Quantidade de alunos inscritos
- Data da incsrição
- Etc

# Processos
## Inscrição
![Alt text](/course-signup-api/img/inscreicao.JPG?raw=true "inscricao")
A inscrição inicia no evento pelo evento "Processar Fila" onde é recuperado do kinessis os dados "por curso". Com cada curso contem uma quantidade finita de alunos possiveis o mesmo pega esta quantidade por ordem de inscrição e gera o comando "Registrar". Os demais alunos entram no expurgo (não detalhado no fluxo)

## Processamento
![Alt text](/course-signup-api/img/processamento-filas.JPG?raw=true "processamento")
O processoamento será executado em thread "paralelizada" via disparo do evento "Registro) onde o mesmo efetua:
- Inscrição do aluno na turma
- Geração das estatisticas
- Disparo do comando "Notificar" aluno

## Notificação
![Alt text](/course-signup-api/img/notificacao.JPG?raw=true "notificacao")
A notificação "inicia" pelo evento "notificação" sendo rsponsavel pelo envio de emails aos alunos. de forma paralelizada (thread). São enviados 3 tipos de email por estre processo:
- Notificação do agendamento
- Aviso de inscrição efetuada com sucesso
- Aviso de inscrição declinada por turma cheia
