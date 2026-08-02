# Cláusulas Contratuais — Infraestrutura e Dados (Nexflow DX)

> **Aviso importante:** este documento é um modelo de referência com boas práticas de transparência sobre infraestrutura, dados e limites de responsabilidade. **Não substitui a revisão de um advogado** — especialmente antes de usar com clientes da União Europeia (GDPR) ou em contratos de maior valor. Os campos marcados como `[DEFINIR]` precisam ser preenchidos antes do uso. Adapte a linguagem conforme a orientação jurídica que receber.

---

## 1. Infraestrutura Compartilhada e Localização dos Dados

### Português

> A infraestrutura utilizada para a prestação dos serviços contratados é hospedada em servidor (VPS) compartilhado, administrado pela Nexflow DX, localizado fisicamente no Brasil. Os dados do Cliente são armazenados em ambiente logicamente isolado dos demais clientes atendidos pela mesma infraestrutura, incluindo banco de dados, instância de automação (n8n) e instância de mensageria (quando aplicável) próprios e exclusivos do Cliente. A infraestrutura não é fisicamente dedicada ao Cliente, salvo quando expressamente contratado o serviço de hospedagem dedicada (ver cláusula 1.1).

**1.1 — Hospedagem Dedicada (opcional)**

> Mediante contratação adicional, o Cliente pode optar por hospedagem dedicada (servidor próprio, não compartilhado com outros clientes), com repasse do custo de infraestrutura acrescido da taxa de gerenciamento da Nexflow DX, conforme proposta comercial específica.

### English

> The infrastructure used to provide the contracted services is hosted on a shared virtual private server (VPS), managed by Nexflow DX, physically located in Brazil. Client data is stored in a logically isolated environment, separate from other clients hosted on the same infrastructure, including a dedicated database, automation instance (n8n), and messaging instance (where applicable) exclusive to the Client. The infrastructure is not physically dedicated to the Client unless dedicated hosting has been expressly contracted (see clause 1.1).

**1.1 — Dedicated Hosting (optional)**

> Upon additional agreement, the Client may opt for dedicated hosting (a server not shared with other clients), with infrastructure costs passed through plus a Nexflow DX management fee, as defined in a specific commercial proposal.

### Español

> La infraestructura utilizada para la prestación de los servicios contratados está alojada en un servidor (VPS) compartido, administrado por Nexflow DX, ubicado físicamente en Brasil. Los datos del Cliente se almacenan en un entorno lógicamente aislado de los demás clientes atendidos por la misma infraestructura, incluyendo base de datos, instancia de automatización (n8n) e instancia de mensajería (cuando corresponda) propias y exclusivas del Cliente. La infraestructura no está físicamente dedicada al Cliente, salvo cuando se contrate expresamente el servicio de alojamiento dedicado (ver cláusula 1.1).

**1.1 — Alojamiento Dedicado (opcional)**

> Mediante contratación adicional, el Cliente puede optar por alojamiento dedicado (servidor propio, no compartido con otros clientes), con el traspaso del costo de infraestructura más la tarifa de gestión de Nexflow DX, conforme a una propuesta comercial específica.

---

## 2. Subprocessadores e Terceiros Envolvidos

### Português

> A prestação dos serviços envolve o uso de fornecedores terceiros ("subprocessadores") para funções específicas de infraestrutura, incluindo, mas não se limitando a: provedor de hospedagem VPS (Hostinger) e provedor de rede, DNS e proteção contra ataques (Cloudflare). Esses fornecedores podem processar metadados de tráfego e conexão, mas não têm acesso direto ao conteúdo das aplicações do Cliente. A Nexflow DX se compromete a informar o Cliente caso haja alteração relevante na lista de subprocessadores utilizados.

### English

> The provision of services involves the use of third-party vendors ("subprocessors") for specific infrastructure functions, including but not limited to: VPS hosting provider (Hostinger) and network, DNS, and attack-protection provider (Cloudflare). These vendors may process connection and traffic metadata but do not have direct access to the content of the Client's applications. Nexflow DX commits to informing the Client of any material change to the list of subprocessors used.

### Español

> La prestación de los servicios implica el uso de proveedores externos ("subprocesadores") para funciones específicas de infraestructura, incluyendo, entre otros: proveedor de alojamiento VPS (Hostinger) y proveedor de red, DNS y protección contra ataques (Cloudflare). Estos proveedores pueden procesar metadatos de tráfico y conexión, pero no tienen acceso directo al contenido de las aplicaciones del Cliente. Nexflow DX se compromete a informar al Cliente sobre cualquier cambio relevante en la lista de subprocesadores utilizados.

---

## 3. Retenção de Dados

### Português

> Os dados do Cliente são mantidos ativos enquanto durar a relação contratual. Backups automáticos dos dados são realizados diariamente e retidos por um período de referência de `[DEFINIR — ex: 30 dias]` em armazenamento externo à infraestrutura principal. Em caso de encerramento do contrato, os dados do Cliente serão mantidos por até `[DEFINIR — ex: 15 dias]` após a data de encerramento, para fins de eventual reativação ou exportação, sendo posteriormente excluídos de forma definitiva, salvo solicitação em contrário por escrito.

### English

> Client data is kept active for the duration of the contractual relationship. Automated backups are performed daily and retained for a reference period of `[DEFINE — e.g., 30 days]` in storage external to the primary infrastructure. Upon contract termination, Client data will be retained for up to `[DEFINE — e.g., 15 days]` following the termination date, for potential reactivation or export purposes, after which it will be permanently deleted, unless otherwise requested in writing.

### Español

> Los datos del Cliente se mantienen activos durante toda la relación contractual. Se realizan copias de seguridad automáticas diariamente y se conservan por un período de referencia de `[DEFINIR — ej.: 30 días]` en almacenamiento externo a la infraestructura principal. En caso de finalización del contrato, los datos del Cliente se conservarán hasta `[DEFINIR — ej.: 15 días]` después de la fecha de finalización, con fines de eventual reactivación o exportación, y posteriormente serán eliminados de forma definitiva, salvo solicitud en contrario por escrito.

---

## 4. Clientes da União Europeia (GDPR) e Legislação Brasileira (LGPD)

### Português

> Independentemente da localização do Cliente, o tratamento de dados pessoais realizado pela Nexflow DX observa os princípios da Lei Geral de Proteção de Dados (LGPD — Lei nº 13.709/2018). Adicionalmente, caso o Cliente ou os titulares de dados por ele tratados estejam localizados na União Europeia, aplica-se o seguinte: os dados pessoais tratados no âmbito da prestação dos serviços são armazenados fora do Espaço Econômico Europeu (Brasil). O Cliente declara estar ciente dessa condição e, quando exigido pela legislação aplicável (GDPR), as partes poderão firmar um Acordo de Tratamento de Dados (Data Processing Agreement — DPA) específico, descrevendo as responsabilidades de cada parte quanto ao tratamento de dados pessoais.

### English

> Regardless of the Client's location, personal data processing carried out by Nexflow DX follows the principles of Brazil's General Data Protection Law (LGPD — Law No. 13,709/2018). Additionally, if the Client or the data subjects processed on its behalf are located in the European Union, the following applies: personal data processed in connection with the services is stored outside the European Economic Area (Brazil). The Client acknowledges this condition, and, where required by applicable law (GDPR), the parties may enter into a specific Data Processing Agreement (DPA) describing each party's responsibilities regarding the processing of personal data.

### Español

> Independientemente de la ubicación del Cliente, el tratamiento de datos personales realizado por Nexflow DX sigue los principios de la Ley General de Protección de Datos de Brasil (LGPD — Ley n.º 13.709/2018). Además, si el Cliente o los titulares de los datos tratados en su nombre se encuentran en la Unión Europea, se aplica lo siguiente: los datos personales tratados en el marco de la prestación de los servicios se almacenan fuera del Espacio Económico Europeo (Brasil). El Cliente declara estar al tanto de esta condición y, cuando lo exija la legislación aplicable (GDPR), las partes podrán suscribir un Acuerdo de Tratamiento de Datos (Data Processing Agreement — DPA) específico, describiendo las responsabilidades de cada parte respecto al tratamiento de datos personales.

---

## 5. Segurança da Infraestrutura (resumo para o cliente)

### Português

> A infraestrutura utilizada segue práticas de segurança que incluem: acesso remoto restrito por autenticação criptográfica (sem uso de senha), firewall configurado para bloquear tráfego não autorizado, bloqueio automático de tentativas de acesso indevido, autenticação em dois fatores nos painéis administrativos, certificados de criptografia (HTTPS) válidos em todos os pontos de acesso, autenticação individual em cada aplicação, e rotina de backup diário com testes periódicos de restauração.

### English

> The infrastructure follows security practices that include: remote access restricted to cryptographic key authentication (no password-based access), a firewall configured to block unauthorized traffic, automatic blocking of unauthorized access attempts, two-factor authentication on administrative panels, valid encryption certificates (HTTPS) at all access points, individual authentication for each application, and a daily backup routine with periodic restoration testing.

### Español

> La infraestructura utilizada sigue prácticas de seguridad que incluyen: acceso remoto restringido mediante autenticación criptográfica (sin uso de contraseña), firewall configurado para bloquear tráfico no autorizado, bloqueo automático de intentos de acceso indebido, autenticación de dos factores en los paneles administrativos, certificados de cifrado (HTTPS) válidos en todos los puntos de acceso, autenticación individual en cada aplicación, y rutina de copia de seguridad diaria con pruebas periódicas de restauración.

---

## 6. Notificação de Incidentes de Segurança

### Português

> Em caso de incidente de segurança que comprovadamente comprometa a confidencialidade, integridade ou disponibilidade dos dados do Cliente, a Nexflow DX se compromete a notificar o Cliente em até `[DEFINIR — ex: 48 horas]` a partir da confirmação do incidente, informando a natureza do ocorrido, os dados potencialmente afetados e as medidas adotadas para contenção e correção.

### English

> In the event of a security incident that is confirmed to compromise the confidentiality, integrity, or availability of Client data, Nexflow DX commits to notifying the Client within `[DEFINE — e.g., 48 hours]` of confirming the incident, describing the nature of the event, the data potentially affected, and the measures taken for containment and remediation.

### Español

> En caso de un incidente de seguridad que comprometa comprobadamente la confidencialidad, integridad o disponibilidad de los datos del Cliente, Nexflow DX se compromete a notificar al Cliente dentro de las `[DEFINIR — ej.: 48 horas]` posteriores a la confirmación del incidente, informando la naturaleza de lo ocurrido, los datos potencialmente afectados y las medidas adoptadas para su contención y corrección.

---

## 7. Responsabilidades do Cliente

### Português

> O Cliente é responsável por: (a) garantir que possui base legal adequada para o tratamento de quaisquer dados pessoais inseridos nas automações e integrações contratadas; (b) não inserir, sem aviso prévio e acordo específico, categorias de dados sensíveis (dados de saúde, biometria, informações financeiras completas de cartão de pagamento, entre outras) que exijam medidas de segurança adicionais às previstas neste contrato; (c) manter a confidencialidade de suas próprias credenciais de acesso às ferramentas contratadas.

### English

> The Client is responsible for: (a) ensuring it has an adequate legal basis for processing any personal data entered into the contracted automations and integrations; (b) not submitting, without prior notice and specific agreement, sensitive data categories (health data, biometric data, full payment card information, among others) that require security measures beyond those provided for in this agreement; (c) maintaining the confidentiality of its own access credentials to the contracted tools.

### Español

> El Cliente es responsable de: (a) garantizar que cuenta con una base legal adecuada para el tratamiento de cualquier dato personal ingresado en las automatizaciones e integraciones contratadas; (b) no ingresar, sin previo aviso y acuerdo específico, categorías de datos sensibles (datos de salud, biometría, información financiera completa de tarjetas de pago, entre otros) que requieran medidas de seguridad adicionales a las previstas en este contrato; (c) mantener la confidencialidad de sus propias credenciales de acceso a las herramientas contratadas.

---

## 8. Disponibilidade do Serviço

### Português

> Os serviços são prestados com base em infraestrutura de nuvem de terceiros e melhores esforços de manutenção e monitoramento contínuo. Salvo quando expressamente contratado um Acordo de Nível de Serviço (SLA) específico, a Nexflow DX não garante disponibilidade ininterrupta (uptime) e não se responsabiliza por indisponibilidades decorrentes de falhas no provedor de hospedagem, ataques de terceiros, ou eventos de força maior, comprometendo-se, contudo, a agir prontamente para restabelecer o serviço.

### English

> The services are provided on third-party cloud infrastructure with best-effort maintenance and continuous monitoring. Unless a specific Service Level Agreement (SLA) has been expressly contracted, Nexflow DX does not guarantee uninterrupted availability (uptime) and is not liable for downtime resulting from hosting provider failures, third-party attacks, or force majeure events, while committing to act promptly to restore service.

### Español

> Los servicios se prestan sobre infraestructura de nube de terceros, con mejores esfuerzos de mantenimiento y monitoreo continuo. Salvo que se haya contratado expresamente un Acuerdo de Nivel de Servicio (SLA) específico, Nexflow DX no garantiza disponibilidad ininterrumpida (uptime) y no se responsabiliza por interrupciones derivadas de fallas del proveedor de alojamiento, ataques de terceros o eventos de fuerza mayor, comprometiéndose, no obstante, a actuar con prontitud para restablecer el servicio.

---

## 9. Limitação de Responsabilidade

### Português

> A responsabilidade da Nexflow DX perante o Cliente, decorrente da prestação dos serviços, fica limitada ao valor efetivamente pago pelo Cliente nos `[DEFINIR — ex: últimos 3 meses]` que antecederem o evento gerador do dano, não respondendo a Nexflow DX por lucros cessantes, danos indiretos ou perda de dados não abrangidos pela rotina de backup descrita neste contrato.

### English

> Nexflow DX's liability to the Client arising from the provision of services is limited to the amount effectively paid by the Client in the `[DEFINE — e.g., last 3 months]` preceding the event giving rise to the damage. Nexflow DX shall not be liable for lost profits, indirect damages, or data loss not covered by the backup routine described in this agreement.

### Español

> La responsabilidad de Nexflow DX frente al Cliente, derivada de la prestación de los servicios, se limita al monto efectivamente pagado por el Cliente en los `[DEFINIR — ej.: últimos 3 meses]` anteriores al evento que dio origen al daño. Nexflow DX no será responsable por lucro cesante, daños indirectos o pérdida de datos no cubiertos por la rutina de copia de seguridad descrita en este contrato.

---

## 10. Lei Aplicável e Foro

### Português

> Este contrato é regido pelas leis da República Federativa do Brasil. Fica eleito o foro da comarca de `[DEFINIR — cidade/estado]` para dirimir quaisquer controvérsias oriundas deste contrato, com renúncia expressa a qualquer outro, por mais privilegiado que seja, salvo disposição diversa acordada por escrito entre as partes em razão da localização do Cliente.

### English

> This agreement is governed by the laws of the Federative Republic of Brazil. The parties elect the courts of `[DEFINE — city/state]` to resolve any disputes arising from this agreement, expressly waiving any other jurisdiction, however privileged, unless otherwise agreed in writing between the parties due to the Client's location.

### Español

> Este contrato se rige por las leyes de la República Federativa de Brasil. Se elige el fuero de `[DEFINIR — ciudad/estado]` para resolver cualquier controversia derivada de este contrato, con renuncia expresa a cualquier otro fuero, por más privilegiado que sea, salvo disposición distinta acordada por escrito entre las partes en razón de la ubicación del Cliente.

---

## Checklist antes de usar este modelo

- [ ] Preencher todos os campos `[DEFINIR]` (períodos de retenção, prazo de notificação de incidente, teto de responsabilidade, foro)
- [ ] Revisar com um advogado, especialmente para clientes da União Europeia
- [ ] Confirmar se o cliente exige DPA (Data Processing Agreement) formal separado
- [ ] Verificar se o valor do contrato justifica um seguro de responsabilidade civil profissional (E&O insurance), comum em contratos internacionais de TI
- [ ] Ajustar a cláusula 8 (Disponibilidade) caso um SLA formal seja oferecido como upsell

---

*Modelo de cláusulas revisado em 02/08/2026. Este documento não constitui aconselhamento jurídico. Revisar com um advogado antes do uso em contratos reais, especialmente para clientes da União Europeia ou contratos de maior valor.*
