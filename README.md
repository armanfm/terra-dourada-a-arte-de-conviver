# terra-dourada-a-arte-de-conviver

# Terra Dourada – Arte de Conviver

**Terra Dourada Upgrade** é uma arquitetura blockchain para eleições, decisões e recompensas descentralizadas, combinando:

- Transparência radical  
- ZK-proofs (off-chain)  
- Governança distribuída (multi-admin, meta final)  
- Incentivos automatizados via smart contract  

O objetivo é fornecer um sistema auditável, seguro e escalável, garantindo que resultados sejam verificáveis e imutáveis, sem depender de uma autoridade central.

> **Status**: MVP funcional. Algumas funcionalidades (multi-admin e recompensas totalmente automáticas) fazem parte da meta para a versão final.

---

## Arquitetura e Componentes

### ProgramConfig
Configurações do programa na blockchain:

| Campo         | Tipo    | Função                                   |
|---------------|---------|------------------------------------------|
| is_initialized| bool    | Inicialização da conta                   |
| authority     | address | Endereço autorizado (MVP: único admin)   |
| paused        | bool    | Flag de pausa do programa                |
| reward_amount | uint256 | Valor padrão de recompensa por lote      |

> No MVP, `authority` é único; na versão final, será distribuído entre múltiplos admins.

### BatchAccount
Representa um lote de votação ou decisão:

| Campo        | Tipo    | Função                                         |
|--------------|---------|------------------------------------------------|
| started      | bool    | Lote iniciado                                  |
| finalized    | bool    | Lote concluído                                 |
| firstClaimer | address | Resultado da votação/decisão                   |
| zkProofHash  | bytes32 | Hash da prova ZK validando o resultado          |

### ZK-Proofs
- Verificação **off-chain**; prova gerada externamente e validada via `IVerifier`.  
- **Public Inputs:** `[winner, batchId]` para prevenir replay de provas.  
- **Challenge:** auditores podem submeter prova de desafio e ser recompensados.

### Utils
Funções utilitárias do contrato:
- `generateBatchHash`: cria hash de batch a partir da prova (para testes/auditoria off-chain)  
- `usedProofHashes`: evita reutilização de provas  
- Eventos detalhados permitem auditoria completa.

---

## Fluxo de Submissão de Lote

1. **Preparação off-chain**: geração de `batchId`, `winner` e prova ZK.  
2. **Submissão**: qualquer usuário envia a prova e `batchIdHash` para o contrato.  
3. **Validação on-chain**:
   - Confere se o contrato não está pausado  
   - Garante que o lote não foi finalizado  
   - Verifica se a prova não foi reutilizada (`usedProofHashes`)  
   - Chama `IVerifier.verifyProof` para validar a prova e obter o vencedor  
4. **Finalização**:
   - Atualiza `firstClaimer` e `zkProofHash`  
   - Marca lote como `finalized`  
   - Emite eventos para auditoria e rastreabilidade  
   - **Recompensa automática (MVP):** `transferFrom(admin, winner, rewardAmount)` (depende de saldo e allowance do admin)

---

## Evolução da Versão Final

- **Multi-admin**  
- **Distribuição totalmente automática**  
- **Integração com Golden Storage**  

As provas podem ser armazenadas em:
- Público (IPFS)  
- Privado (Google Cloud, AWS)  
- Híbrido

> Para casos eleitorais, a prova final fica restrita ao TSE, garantindo sigilo e conformidade legal.

**Sigilo adicional:**  
O sistema permite que o registro de cada voto e a prova criptográfica correspondente sejam mantidos em módulos isolados, de forma que apenas o TSE (ou órgão autorizado) tenha acesso à prova final. A blockchain registra apenas o **hash verificável**, sem vínculo com o eleitor.

---

## Governança Descentralizada

- **Meta final**: multi-admin distribuído, exigindo múltiplas aprovações para submissão de lote.  
- **MVP**: único admin, mas arquitetura pronta para evolução.  
- **Transparência**: todas as alterações e finalizações de lote registradas on-chain.

---

## Casos de Uso

### Sistema Eleitoral com Transparência Radical
- Cada município processa lotes em mini-rollups  
- Rollup estadual agrega resultados municipais  
- Rollup nacional consolida resultados estaduais  
- Prova final publicada na blockchain, auditável e confiável

**Incentivos:**
- Competição de validadores para gerar ZK-proofs rapidamente  
- Recompensas premium  
- Penalidades automáticas (slashing)  
- Auditoria aberta com recompensa via challenge ZK-proof

- **Versatilidade:**  
> Embora o exemplo principal seja um sistema eleitoral, a arquitetura modular do Terra Dourada permite aplicação em outros cenários que demandem confiança criptográfica e auditabilidade.

---

## Glossário

- **Admin**: responsável pelo contrato no MVP  
- **Multi-admin**: múltiplos administradores  
- **ZK-Proof**: prova criptográfica sem revelar dados  
- **Batch**: lote de votação ou decisão  
- **Golden Storage**: repositório off-chain de provas ZK  
- **Storage Público/Privado/Híbrido**: IPFS, Google Cloud, AWS, TSE  
- **approve / allowance**: permissões ERC-20 usadas no MVP  
- **Instituições regulatórias**: CVM, TSE, órgãos financeiros ou de auditoria  

---

## Conclusão

**Terra Dourada Upgrade** fornece um framework universal para:

- Processos decisórios verificáveis criptograficamente  
- Sistemas eleitorais auditáveis por terceiros  
- Mecanismos de governança com transparência radical  
- Compliance regulatório via proof-of-integrity  

O MVP já oferece registro de batches, verificação off-chain de provas e recompensas automáticas.  
A versão final trará multi-admin, distribuição automática e integração completa com Golden Storage.


O MVP já entrega registro de batches, verificação off-chain de provas e recompensas automáticas limitadas pelo approve, enquanto a versão final evoluirá para multi-admin, distribuição totalmente automática e integração completa de Golden Storage, com sigilo adequado às instituições envolvidas.

A arquitetura, por ser flexível, permite adaptação para diferentes setores: processos eleitorais, mercados financeiros, auditorias de dados ou qualquer cenário que demande confiança criptográfica e prova de integridade.
