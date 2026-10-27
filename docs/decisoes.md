# Decisões e aprendizados

## 1. Mostrar, nunca calcular
O portal não calcula juros, não soma valores e não monta Pix. Ele mostra o que o ERP e o banco geraram, depois de conferir. Assim o portal não cria uma segunda "verdade" financeira, e a superfície para fraude diminui.

## 2. Duas opiniões sobre cada pagamento
Mesmo com a validação no portal, um serviço independente (o vigia) refaz a conferência como se fosse um cliente. Se o portal for comprometido, o vigia percebe e trava os pagamentos. Se o vigia sair do ar, o portal trava sozinho (falha fechada).

## 3. Alerta é produto
No começo, toda falha ia para o Telegram do dono, e isso não se sustenta. O desenho final:
- **Urgente (dono):** só invasão, adulteração e o "bom dia, está tudo no ar".
- **Suporte (equipe):** falhas, erros de tela com código, cobranças para validar.
- Limite global por janela de tempo, para nunca inundar o grupo.

## 4. Erro com código
Quando algo dá errado de verdade, o cliente vê um mascote, a frase "Ops! Algo não saiu como esperado", um código `JM-XXXX` e um único botão "Falar com o suporte", que já leva o código. Queda de internet do próprio cliente **não** abre essa página: só mostra um aviso no lugar.

## 5. Ritual para publicar código de pagamento
1. Calcular o hash da versão nova.
2. O vigia passa a aceitar o hash atual e o novo.
3. Publicar.
4. Conferir a saúde do portal (versão, hash, pagamentos liberados).
5. O vigia passa a aceitar só o hash novo.

## 6. Rede de segurança com data para acabar
Ao separar os tokens do ERP, alguma permissão podia faltar. Por isso, uma consulta recusada no token de leitura é refeita pelo de gravação (só consulta, nunca em tabela sensível) e gera um aviso diário. Os avisos mostram o que liberar. Depois de alguns dias sem aviso, a rede é desligada.

## 7. Pequenos detalhes que evitam incidentes
- Um comentário `//` no meio de uma linha de código engoliu o resto da linha. Os testes pegaram o erro antes de ir para produção.
- Uma rota interna exigia um cabeçalho que o vigia não mandava, e a conferência nunca rodava. Foi descoberto pelo log de cada conferência e corrigido.
- O cálculo do IP do cliente precisa considerar o proxy da nuvem, senão todos os clientes "têm o mesmo IP" e os limites se tornam injustos.
