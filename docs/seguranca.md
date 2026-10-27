# Segurança

Revisão feita com base no **OWASP Top 10:2025**, com testes em ambiente local (ERP simulado, nada toca dados reais) e validação em produção, sem testes ofensivos contra sistemas de terceiros.

> Os valores exatos de limites, janelas e regras internas foram omitidos de propósito.

## Mapa OWASP → o que foi feito

| Categoria OWASP 2025 | Medidas no projeto |
|---|---|
| **A01 Controle de acesso** | A fatura só é servida ao dono da sessão. Fatura paga, de outro cliente ou com ID disfarçado é recusada em todas as rotas. A ponte só aceita uma lista fechada de consultas por sistema. Os tokens do ERP são separados: leitura e gravação restrita. |
| **A02 Configuração** | Segredos só em variáveis de ambiente. Cabeçalhos de segurança (CSP, HSTS, frame-ancestors). Rotas de diagnóstico e autoteste removidas antes do lançamento. |
| **A03 Cadeia de suprimentos** | Poucas dependências. Fontes e scripts servidos pelo próprio portal, sem CDN de terceiros. Alerta quando o CI ou o código sensível muda. |
| **A04 Criptografia** | HTTPS em tudo. Hash do código de pagamento aprovado e conferido pelo vigia. |
| **A05 Injeção** | O corpo da consulta ao ERP é sempre remontado pela ponte com campos fixos. Query string descartada. Entradas validadas por formato. |
| **A06 Design inseguro** | O portal não calcula valor nem gera Pix. Trava em falha fechada. Desbloqueio de confiança com regras e cotas. |
| **A07 Autenticação** | Resposta genérica para CPF inexistente e senha errada. Limites por aparelho e por IP. Bloqueio de quem testa muitos CPFs. Proteção contra varredura distribuída. |
| **A08 Integridade** | Pix (BR Code: CRC, PSP, URL) e boleto (banco, dígitos verificadores) conferidos antes de exibir. Vigia como verificador independente. |
| **A09 Log e alerta** | Alertas separados: segurança vai para o dono, falhas vão para o suporte. Limite global de alertas. Sem dado pessoal em log. |
| **A10 Erros e exceções** | Página de erro com código rastreável. Erros do servidor nunca vazam detalhes ao cliente. |

## ERP: menor privilégio na prática

- **Usuário de leitura:** só *acesso + menu + listar* nas tabelas que o portal usa, e nenhum botão de gravação.
- **Usuário de gravação:** exclusão bloqueada em todas as telas. Financeiro, configuração, usuários e permissões bloqueados. Continua liberado só o necessário (cliente, contrato, ordem de serviço, Pix, boleto, desbloqueio).
- **Conferência final:** 2.283 permissões verificadas, 0 divergências.
- **Lição aprendida:** no ERP, a primeira alteração numa tela sem regra própria cria a regra com acesso **bloqueado**. Sempre conferir o acesso depois de mudar uma permissão.

## O que foi testado (ambiente local, ERP simulado)

**Jornada com 30 verificações:**
- senha errada e CPF inexistente com a mesma resposta genérica;
- dados pessoais mínimos (primeiro nome, documento mascarado);
- Pix e boleto de fatura de outro cliente recusados;
- ID de fatura disfarçado recusado;
- Pix com recebedor ou domínio estranho e boleto de outro banco ou adulterado **não aparecem**;
- desbloqueio repetido não libera de novo;
- sessão antiga não vale depois de sair.

**8 cenários de ataque, todos barrados:**
- robô insistindo é bloqueado, e um cliente legítimo em outro IP continua entrando;
- credencial errada na rota interna é bloqueada, e o vigia legítimo continua conferindo;
- varredura distribuída (muitos IPs) liga a proteção automática: o visitante vê "Já voltamos" e nenhuma consulta chega ao ERP.
