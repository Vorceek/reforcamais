# Infraestrutura de Deploy / Publicação - Reforça+

## Escolha
Arquitetura **cloud, serverless/PaaS, custo zero**, composta por três serviços gerenciados:

| Componente | Serviço | Plano | Função |
|---|---|---|---|
| Frontend PWA | **Vercel** | Hobby (gratuito) | Hospedagem estática com CDN global, HTTPS automático, preview por PR |
| API REST | **Render** | Web Service Free | Container Docker Node.js, deploy automático a partir do GitHub |
| Banco de dados | **Neon** | Free | PostgreSQL serverless, 0,5 GB, branching de banco |

Serviços de apoio: GitHub (código, CI/CD com Actions), Firebase Cloud Messaging (push, gratuito), Sentry e UptimeRobot (monitoramento, planos gratuitos).

## Justificativa

### 1. Custo zero e sustentabilidade do projeto extensionista
O projeto atende uma comunidade sem orçamento para infraestrutura. Todos os serviços escolhidos têm planos gratuitos permanentes (não apenas créditos temporários), o que permite que a solução continue no ar após o encerramento da disciplina, sem depender de cartão de crédito ou renovação de créditos.

### 2. HTTPS obrigatório para PWA
Service workers, instalação na tela inicial e notificações push **só funcionam sobre HTTPS**. Vercel e Render fornecem certificados TLS automáticos (Let's Encrypt), sem configuração manual - em um VPS self-host isso exigiria gerenciar Nginx, Certbot e renovação.

### 3. Aderência ao DevOps
Os três serviços integram-se nativamente ao GitHub: cada `push` na branch `main` gera deploy automático, e cada Pull Request gera um ambiente de *preview* isolado na Vercel. O Neon permite criar *branches* de banco para testes, alinhado ao pipeline de CI/CD modelado no diagrama DevOps.

### 4. Baixa carga operacional para uma equipe pequena
Não há servidor para atualizar, firewall para configurar nem backups manuais: o provedor cuida de SO, patches, escalabilidade e disponibilidade. Isso libera a equipe para focar no produto e na comunidade.

## Comparação com alternativas

| Alternativa | Por que não foi escolhida |
|---|---|
| **AWS / Azure / GCP (free tier)** | Free tier expira em 12 meses ou depende de créditos estudantis; exige cartão de crédito; curva de aprendizado (IAM, VPC, EC2) desproporcional ao tamanho do projeto; risco de cobranças inesperadas. |
| **Self-host (VPS DigitalOcean/Hetzner ou servidor da instituição)** | Custo mensal recorrente (VPS) ou dependência de aprovação/manutenção do setor de TI (servidor institucional); toda a operação (SO, TLS, backup, monitoramento) fica com a equipe; menor disponibilidade. |
| **Firebase (Hosting + Firestore + Functions)** | Banco NoSQL menos adequado ao modelo relacional (alunos, tutores, aulas, frequências); *lock-in* forte com o Google; Functions exige plano Blaze (pago) para chamadas externas. |
| **Heroku** | Plano gratuito foi descontinuado em 2022. |
| **Supabase (Postgres + Auth)** | Alternativa válida e equivalente ao Neon; Neon foi preferido por ser exclusivamente PostgreSQL, sem projetos pausados após 1 semana de inatividade no plano free, e por oferecer branching de banco. Pode ser substituído sem alterar a arquitetura. |

## Limitações conhecidas e mitigações
- **Render Free "dorme" após 15 min sem tráfego** (cold start ~30 s). Mitigação: ping periódico via UptimeRobot e cache offline no PWA, que mantém a interface utilizável mesmo com a API lenta.
- **Neon Free: 0,5 GB.** Suficiente para milhares de registros textuais; materiais pesados (PDF/vídeo) ficam em links externos ou Cloudinary (free).
- **Portabilidade:** a API é empacotada em Docker e o banco é PostgreSQL padrão, permitindo migrar para VPS ou outra cloud sem reescrita.

## Ativação da infraestrutura (checklist)
- [ ] Criar conta no GitHub e repositório público `reforcamais`
- [ ] Criar conta na Vercel com login GitHub e importar o repositório (pasta `frontend/`)
- [ ] Criar conta no Render com login GitHub e criar Web Service (pasta `backend/`, Dockerfile)
- [ ] Criar conta no Neon, criar projeto `reforcamais` e copiar a `DATABASE_URL`
- [ ] Configurar variáveis de ambiente (`DATABASE_URL`, `JWT_SECRET`, `FCM_KEY`) no Render e `VITE_API_URL` na Vercel
- [ ] Criar projeto no Firebase apenas para Cloud Messaging
- [ ] Configurar UptimeRobot e Sentry (opcional)
