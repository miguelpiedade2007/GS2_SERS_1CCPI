# Testes com usuários – módulo de energia

## Objetivo
Verificar se as orientações sobre energia no banho são **compreendidas** e **úteis** para o público do Citizen Assist AI, e ajustar o assistente a partir do que os participantes disserem.

## Protocolo
- **Participantes:** 5 a 8 pessoas de perfis diferentes: inquilino, proprietário de casa, pessoa idosa, família com Tarifa Social e jovem que divide a conta.
- **Privacidade:** cada participante é identificado só por um código (P1, P2...). O assistente não pede nome, endereço, CPF ou telefone, e o registro apaga automaticamente padrões de CPF, telefone e e-mail digitados por engano.
- **Como registrar:** os testes foram acompanhados presencialmente e registrados na tabela abaixo. O assistente também tem um registro automático: rodar `python -m app.servidor`, marcar **"Modo teste com usuários"** na página e, depois de cada resposta, clicar em *Sim*, *Entendi, mas não ajudou* ou *Não entendi*. Essas avaliações são gravadas em `testes/registro_feedback.csv`, criado automaticamente na primeira avaliação. No terminal (`python -m app.cli`), use o comando `avaliar`.
- **Duração:** cerca de 10 minutos por participante.

## Situações testadas

| Código | Tarefa dada ao participante | O que se observa |
|---|---|---|
| T1 | "Descubra quanto o chuveiro gasta por mês na sua casa." | Se informa pessoas, tempo de banho ou potência e entende o valor em kWh e em R$ |
| T2 | "Peça dicas para economizar sem gastar dinheiro." | Se entende o banho consciente e a posição verão |
| T3 | "Descubra se vale a pena um aquecedor solar." | Se entende investimento, retorno e as condições (telhado, orçamentos) |
| T4 | "Compare o aquecedor solar com a placa solar." | Se percebe a diferença entre aquecer água e gerar eletricidade e o efeito no horário de pico |
| T5 | "Sua família tem Tarifa Social. Pergunte o que muda." | Se entende a franquia de 80 kWh e o recálculo |
| T6 | "Pergunte de onde vieram os números e se são garantidos." | Se encontra as fontes e as limitações |

Perguntas finais (escala de 1 a 5):
1. As respostas foram fáceis de entender?
2. Você saberia o que fazer na sua casa depois da conversa?
3. Você confiou nos números apresentados? Por quê?

## Registro dos testes com usuários

| Participante | Perfil | Situação | Entendeu? | Útil? | Comentário (resumido) | Ajuste realizado |
|---|---|---|---|---|---|---|
| P1 |Inquilino (Aluguel) | T1, T2, T3 | Sim | Sim |Viu rápido que o aquecedor solar não compensa; vai usar chave verão e reduzir o tempo | As respostas sobre aquecedor solar e placa solar passaram a começar avisando que a obra exige imóvel próprio ou autorização do proprietário (ou do condomínio), e indicam o banho consciente para quem mora de aluguel |
| P2 |Proprietário de casa |T1, T2, T4 |Sim |Sim |Entendeu a diferença entre aquecer água e gerar eletricidade, além do efeito no horário de pico | Nenhum ajuste necessário |
| P3 |Pessoa idosa |T1, T2, T6 | Parcialmente|Sim | Adicionar breve explicação em linguagem cotidiana | Interface simplificada: só 4 perguntas principais ficam à vista (as outras foram para "Mais perguntas"), e a mensagem de boas-vindas passou a explicar o que o assistente faz, com exemplos de perguntas |
| P4 |Família com tarifa social |T1, T2, T5 |Sim |Sim |Compreendeu a franquia de 80kWh | Nenhum ajuste necessário; regra da franquia funcionou conforme esperado |
| P5 |Jovem que divide a conta |T1, T6 |Sim | Sim | Usou o cálculo do chuveiro para conversar sobre a divisão da conta e conferiu as fórmulas e as fontes | Nenhum ajuste necessário |

## Testes internos e ajustes realizados

Antes dos testes com o público, as perguntas acima foram simuladas no assistente e o seguinte foi corrigido:

| Situação | Problema encontrado | Ajuste realizado |
|---|---|---|
| "Aquecedor solar vale a pena?" | A expressão genérica "vale a pena" fazia o assistente responder com a comparação geral, e não sobre o aquecedor | Assuntos específicos (aquecedor solar, placa solar, Tarifa Social) passaram a pesar mais na identificação da intenção |
| "Comparar aquecedor solar e placa solar" | A pergunta citava duas tecnologias e caía em uma só | Quando as duas aparecem juntas, o assistente responde com a comparação |
| Resposta sobre a placa solar | A potência aparecia como "1.10 kWp", no formato americano | Números passaram a usar vírgula decimal ("1,10 kWp") |
| Família com Tarifa Social | A economia em reais era calculada como se todo kWh fosse cobrado | Implementada a regra da franquia de 80 kWh: só o excedente vira economia em reais, e o assistente avisa isso |
| Comparação com Tarifa Social | O texto dizia sempre que a fotovoltaica tem o retorno mais rápido, o que nem sempre vale | A recomendação passou a comparar os retornos calculados para cada família |
| Potência informada como "5,5 kW" | Só "W" era reconhecido | Passou a aceitar kW e W |

## Revisão técnica e ajustes realizados

Uma revisão técnica cruzada das respostas apontou incoerências entre a análise e o assistente. Foi corrigido o seguinte:

| Situação testada | Problema encontrado | Ajuste realizado |
|---|---|---|
| "5 pessoas, banho de 12 minutos, chuveiro de 7500 W" e depois "Como você calculou?" | O consumo era calculado por P × t (225 kWh), mas a explicação mostrava o método físico com 5.500 W (≈ 167 kWh) | A explicação passou a usar o mesmo método e os mesmos dados do resultado; há um teste automático para isso |
| Banho consciente | A redução da temperatura da água no verão só era aplicada à alternativa A, o que favorecia artificialmente a comparação | Todas as alternativas passaram a usar a mesma temperatura de água e de banho. A ficou só com a redução do tempo (30%), e a posição verão virou dica adicional com hipótese explícita, fora da comparação |
| "Temos tarifa social e consumo de 150 kWh" | O assistente dizia que o chuveiro custava R$ 99,94 "na conta", ignorando a franquia | Passou a separar o custo equivalente sem benefício do valor efetivamente cobrado: (150 − 80) × R$ 0,90 = R$ 63,00 |
| "10 pessoas" e depois "aquecedor solar" | O aquecedor seguia com 200 L, 2 m² e R$ 5.000 para qualquer família | O aquecedor passou a ser dimensionado pela água quente usada por dia (volume, área e custo), com aviso acima de 1.000 L |
| Fotovoltaica na tabela de resultados | Aparecia "0 kWh da rede", embora o chuveiro continue consumindo à noite | Foram separados consumo físico, energia evitada e energia compensada. A compensação passou a ser limitada pelo consumo da casa e pelo custo de disponibilidade, e o crédito de 85% passou a ser descrito como hipótese do autor, não como valor da lei |
| "Banho de 0 minutos", "0 banhos por dia" | O assistente quebrava (divisão por zero) | Os valores fora da faixa são recusados e o assistente pede um valor válido |
| Documentos × código | README e análise podiam ficar com números diferentes dos calculados | As tabelas passaram a ser geradas pelo script em todos os documentos, e um teste falha se houver divergência |
| Premissas sem fonte | Potência e área do módulo estavam como dado público sem referência, e a irradiação não indicava local nem inclinação | Passaram a ser estimativas justificadas, com local, inclinação e faixa da irradiação registrados |
