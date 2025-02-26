
<h1>📌 Documentação - Automação de Desligamento e Inicialização de EC2 com AWS Lambda</h1>

<h2>1. Visão Geral</h2>
<p>Este documento descreve o funcionamento de uma <strong>função AWS Lambda</strong> que permite iniciar e desligar instâncias EC2 com base em eventos programados no <strong>AWS EventBridge</strong>.</p>

<h2>2. Código da Função Lambda</h2>
<p>O código abaixo recebe eventos contendo os IDs das instâncias e a ação desejada (<code>start</code> ou <code>stop</code>) e executa a operação correspondente utilizando a API do AWS Boto3:</p>

<pre><code>
import boto3
region = boto3.session.Session().region_name
ec2_client = boto3.client('ec2', region_name=region)

def lambda_handler(event, context):
    action = event.get('action')
    instanceIds = event.get('instanceIds')
    if action == 'start':
        ec2_client.start_instances(InstanceIds=instanceIds)
        print(f'Instâncias {instanceIds} foram iniciadas.')
    elif action == 'stop':
        ec2_client.stop_instances(InstanceIds=instanceIds)
        print(f'Instâncias {instanceIds} foram desligadas.')
</code></pre>

<h2>3. Configuração de Permissões (IAM Role para Lambda)</h2>

<h3>Passo 1 - Criar a Role</h3>
<ol>
    <li>Acesse o <strong>AWS IAM Console</strong>.</li>
    <li>Vá para <strong>Roles</strong> → <strong>Create Role</strong>.</li>
    <li>Escolha <strong>AWS Service</strong> e selecione <strong>Lambda</strong>.</li>
    <li>Clique em <strong>Next</strong> para adicionar permissões.</li>
</ol>

<h3>Passo 2 - Criar a Política de Permissão</h3>
<pre><code>
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances",
                "ec2:DescribeInstances"
            ],
            "Resource": "arn:aws:ec2:us-east-1:YOUR_AWS_ACCOUNT_ID:instance/*"
        }
    ]
}
</code></pre>

<p>Substitua <code>YOUR_AWS_ACCOUNT_ID</code> pelo ID da sua conta AWS.</p>

<h3>Passo 3 - Associar a Política à Role</h3>
<ol>
    <li>Volte à criação da Role.</li>
    <li>Anexe a política <code>Lambda_EC2_StartStop_Policy</code>.</li>
    <li>Finalize e nomeie a Role como <code>Lambda_EC2_StartStop_Role</code>.</li>
    <li>Associe essa Role à função Lambda.</li>
</ol>

<h2>4. Configuração do AWS EventBridge (Agendamento de Desligamento e Ligamento Automático)</h2>

<h3>Passo 1 - Criar uma Regra no EventBridge</h3>
<ol>
    <li>Acesse <strong>AWS EventBridge</strong>.</li>
    <li>Vá para <strong>Rules</strong> → <strong>Create Rule</strong>.</li>
    <li>Dê um nome para a regra (<code>Stop_EC2_Weekend</code>).</li>
    <li>Escolha <strong>Schedule</strong> como o tipo de regra.</li>
</ol>

<h3>Passo 2 - Configuração do Cron Job</h3>
<p>O cron do AWS segue o <strong>UTC (Horário de Greenwich)</strong>, enquanto o <strong>horário de Brasília (BRT)</strong> está normalmente:</p>
<ul>
    <li><strong>UTC-3</strong> no horário normal.</li>
    <li><strong>UTC-2</strong> durante o horário de verão (se houver).</li>
</ul>

<h3>Exemplos de agendamento:</h3>
<table border="1">
    <tr>
        <th>Ação</th>
        <th>Horário (BRT)</th>
        <th>Horário (UTC)</th>
        <th>Expressão Cron</th>
    </tr>
    <tr>
        <td><strong>Desligar EC2 (Sexta às 19h BRT)</strong></td>
        <td>19:00 BRT</td>
        <td>22:00 UTC</td>
        <td><code>0 22 * * 5</code></td>
    </tr>
    <tr>
        <td><strong>Ligar EC2 (Segunda às 08h BRT)</strong></td>
        <td>08:00 BRT</td>
        <td>11:00 UTC</td>
        <td><code>0 11 * * 1</code></td>
    </tr>
</table>

<h3>Passo 3 - Configuração da Regra</h3>
<pre><code>
{
    "action": "stop",
    "instanceIds": ["i-xxxxxxxxxxxxxxxxx"]
}
</code></pre>

<h2>5. Testando a Automação</h2>
<ol>
    <li>No console do <strong>AWS Lambda</strong>, vá até sua função.</li>
    <li>Clique em <strong>Test</strong> e insira um payload de teste.</li>
    <li>Execute e verifique se as instâncias foram desligadas corretamente.</li>
</ol>

<h2>6. Conclusão</h2>
<p>Agora, sua infraestrutura EC2 será desligada automaticamente nos finais de semana e religada nos dias úteis, reduzindo custos operacionais sem comprometer a produtividade.</p>

<p>Se precisar ajustar os horários, basta editar as regras do <strong>AWS EventBridge</strong>. 🚀</p>

<p>Referência: https://www.youtube.com/watch?v=9ac8bCPHPlQ&list=PLXy4Oyd2UstrBhHnwiM3VI0lXSgLyo6Sn<p>
</body>
</html>
