# Helm and ArgoCD Learning Labs 🚀

A comprehensive, hands-on curriculum for learning **Helm** (Kubernetes package manager) and **ArgoCD** (GitOps continuous deployment) from fundamentals to production-grade deployments.

**14 modules • 56 files • 365+ KB of learning content**

## 📚 Curriculum Overview

| Module | Topic | Focus | Duration |
|--------|-------|-------|----------|
| **00** | Setup & Prerequisites | Environment preparation | 30 min |
| **01** | Helm Basics | Charts, templates, CLI | 2 hours |
| **02** | Helm Chart Design | Production structure | 2 hours |
| **03** | Helm Templating & Functions | Advanced templating | 2 hours |
| **04** | Helm Values & Environments | Multi-environment setup | 2 hours |
| **05** | Helm Testing & Linting | Quality assurance | 1.5 hours |
| **06** | ArgoCD Basics | Installation & fundamentals | 2 hours |
| **07** | ArgoCD Applications & Sync | Deployment strategies | 2 hours |
| **08** | GitOps Workflows & Promotion | Multi-environment promotion | 2.5 hours |
| **09** | App-of-Apps Pattern | Hierarchical deployments | 2 hours |
| **10** | ArgoCD Projects, RBAC & Multi-Tenancy | Enterprise features | 2 hours |
| **11** | ArgoCD Secrets & Image Updates | Security & automation | 2 hours |
| **12** | Argo Rollouts & Progressive Delivery | Advanced deployment | 2.5 hours |
| **13** | Troubleshooting & Best Practices | Production readiness | 2 hours |
| **14** | End-to-End GitOps Labs | Capstone project | 3 hours |

**Total learning time:** ~30+ hours (self-paced)

## 🎯 What You'll Learn

### By End of Module 5 (Helm Mastery)
- ✅ Create production-ready Helm charts
- ✅ Template Kubernetes manifests dynamically
- ✅ Manage multi-environment configurations
- ✅ Test and validate charts
- ✅ Integrate with CI/CD pipelines

### By End of Module 11 (ArgoCD Advanced)
- ✅ Set up GitOps workflows
- ✅ Implement multi-environment promotion
- ✅ Configure RBAC and multi-tenancy
- ✅ Manage secrets securely
- ✅ Automate image updates

### By End of Module 14 (Production Ready)
- ✅ Deploy complete application stacks
- ✅ Implement canary and blue-green deployments
- ✅ Monitor and troubleshoot production systems
- ✅ Design disaster recovery procedures
- ✅ Implement GitOps best practices

## 📖 Module Structure

Each module contains 4 files:

### 1. **README.md** - Learning Guide
- What you'll learn
- Prerequisites
- Key concepts
- Hands-on lab with commands
- Validation checklist
- Common mistakes
- Troubleshooting tips
- Next steps

### 2. **exercises.md** - Practice Problems
- 10 exercises (easy → medium progression)
- Real-world scenarios
- Self-paced challenges

### 3. **solutions.md** - Detailed Solutions
- Complete bash commands
- Explanations for each step
- Best practices highlighted

### 4. **cheatsheet.md** - Quick Reference
- Command tables
- Configuration templates
- Common patterns

## 🚀 Getting Started

### Prerequisites
- Linux/Mac/Windows with WSL2
- Docker or container runtime
- Minikube or Kind (local Kubernetes)
- Git
- kubectl, Helm 3.x installed
- Text editor (VS Code recommended)

### Installation (5 minutes)

```bash
# Clone repository
git clone https://github.com/your-org/helm-and-argocd-labs.git
cd helm-and-argocd-labs

# Start with module 00
cd 00-setup-and-prerequisites
cat README.md

# Follow the hands-on lab section
# Complete exercises
# Review solutions
# Use cheatsheet as reference
```

### Recommended Learning Path

**Week 1-2: Helm Fundamentals**
```
00-setup-and-prerequisites
  ↓
01-helm-basics
  ↓
02-helm-chart-design
  ↓
03-helm-templating-and-functions
  ↓
04-helm-values-and-environments
  ↓
05-helm-testing-and-linting
```

**Week 3-4: ArgoCD & GitOps**
```
06-argocd-basics
  ↓
07-argocd-applications-and-sync
  ↓
08-gitops-workflows-and-promotion
  ↓
09-app-of-apps-pattern
```

**Week 5-6: Enterprise & Production**
```
10-argocd-projects-rbac-and-multi-tenancy
  ↓
11-argocd-secrets-and-image-updates
  ↓
12-argo-rollouts-and-progressive-delivery
  ↓
13-troubleshooting-and-best-practices
```

**Week 7: Capstone**
```
14-end-to-end-gitops-labs
  → Complete real-world project
  → Deploy full application stack
  → Implement all learned patterns
```

## 📋 How to Use Each Module

### Step 1: Read README.md
Start with the overview section to understand what you'll learn.

```bash
cd 01-helm-basics
cat README.md
```

### Step 2: Follow the Hands-on Lab
Execute commands from the "Hands-on Lab" section step-by-step:

```bash
# Copy commands from README
kubectl apply -f ...
helm create myapp
# ... continue through all steps
```

### Step 3: Complete Exercises
Work through all 10 exercises in exercises.md:

```bash
cat exercises.md
# Complete Exercise 1, 2, 3... at your own pace
```

### Step 4: Check Solutions
Compare your answers with solutions.md:

```bash
cat solutions.md
# Review detailed explanations and commands
```

### Step 5: Use Cheatsheet
Keep cheatsheet.md open as reference while practicing:

```bash
cat cheatsheet.md
# Quick lookup for commands and patterns
```

## 🎓 Learning Tips

### For Beginners
- **Take time on Helm modules (00-05)** - Foundation is crucial
- **Run every command yourself** - Don't just read, execute
- **Modify examples** - Change values, experiment with templates
- **Take notes** - Write your own explanations

### For Experienced Users
- **Skip module 00 if comfortable** - Check prerequisites
- **Focus on ArgoCD modules (06-11)** - Most enterprise value
- **Advanced patterns (12-14)** - Production techniques
- **Use as reference** - Cheatsheets are comprehensive

### Best Practices
- Create a dedicated folder for labs
- Use version control for your changes
- Document what you learn
- Test in dev before production patterns
- Join community discussions

## 🔧 Tools & Technologies

### Core Technologies
- **Kubernetes** - Container orchestration
- **Helm 3.x** - Package manager
- **ArgoCD** - GitOps CD platform
- **Argo Rollouts** - Progressive delivery
- **Docker** - Container images
- **Git** - Source control

### Optional But Recommended
- Prometheus - Metrics collection
- Grafana - Visualization
- Istio/Flagger - Service mesh
- Sealed Secrets - Encryption
- GitHub/GitLab - Git hosting

## 📊 Real-World Applications

These labs teach you patterns used in:
- ✅ SaaS platforms (multi-tenant deployment)
- ✅ Microservices architecture (app-of-apps)
- ✅ Multi-region deployments (GitOps at scale)
- ✅ Compliance-heavy industries (audit trails via Git)
- ✅ DevOps/Platform Engineering teams

## 🏆 Completion Milestones

Track your progress:

- [ ] **Helm Proficiency** - Complete modules 00-05
- [ ] **GitOps Foundation** - Complete modules 06-09
- [ ] **Enterprise Ready** - Complete modules 10-11
- [ ] **Advanced Patterns** - Complete modules 12-13
- [ ] **Production Master** - Complete module 14

## 🐛 Troubleshooting

### Common Issues & Solutions

**Q: "Command not found: kubectl"**
- Verify installation: `kubectl version`
- Add to PATH if needed

**Q: "Failed to create pod"**
- Check cluster running: `kubectl cluster-info`
- Check resources: `kubectl top nodes`

**Q: "Image pull backoff"**
- Verify image exists: `docker pull image:tag`
- Check registry credentials

**Q: "Connection refused"**
- Verify port-forwarding: `kubectl port-forward ...`
- Check firewall rules

## 📚 Additional Resources

### Documentation
- [Helm Official Docs](https://helm.sh/docs/)
- [ArgoCD Official Docs](https://argo-cd.readthedocs.io/)
- [Kubernetes Docs](https://kubernetes.io/docs/)

### Community
- Join DevOps/Kubernetes communities
- Participate in Helm/ArgoCD discussions
- Share your labs and projects

## 💡 Course Outcomes

After completing this curriculum, you'll be able to:

1. **Design Helm charts** from scratch with best practices
2. **Deploy applications** using GitOps principles
3. **Manage multiple environments** with consistent configurations
4. **Implement canary deployments** with automated rollbacks
5. **Troubleshoot production issues** systematically
6. **Architect multi-tenant systems** with proper isolation
7. **Automate image updates** with security scanning
8. **Design disaster recovery** procedures
9. **Lead DevOps initiatives** in your organization

## 📝 License

Open source - Free to use, modify, and distribute

## 🤝 Contributing

Found issues or improvements?
- Report bugs clearly
- Suggest enhancements
- Share your solutions
- Help others in the community

---

## Quick Start

```bash
# Clone and start learning
git clone <repo-url>
cd helm-and-argocd-labs
cd 00-setup-and-prerequisites

# Read, execute, learn
cat README.md          # Understand concepts
cat exercises.md       # Practice
cat solutions.md       # Verify
cat cheatsheet.md      # Reference

# Move to next module
cd ../01-helm-basics
# Repeat the process
```

**Happy Learning! 🎉**

For questions or feedback, check the discussion section or documentation index.
