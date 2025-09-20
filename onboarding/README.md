# Onboarding Documentation

## ⚠️ Under Construction - Completely New Content

**This entire onboarding directory (15 files) is NEW CONTENT** not present in the original Developer Machine Setup 3.0 document. Specifically:

### NEW FILES (Not in Original):
- Company Context, Architecture Overview, Troubleshooting Skills
- First Week Schedule, Learning Milestones, Buddy Assignment  
- Onboarding Manager Guide, Checklist, Production Access SOP
- All remaining 6 files in this directory

### What's Enhanced:
- Enterprise-level developer onboarding process
- Structured 30/60/90 day learning milestones
- Manager guides and buddy system
- Comprehensive troubleshooting methodology

**Original Document Coverage:** None - this represents a complete expansion beyond technical setup into human development processes.

## Overview

This directory contains comprehensive onboarding documentation designed to develop independent problem solvers who understand Method's platform and can troubleshoot effectively. The onboarding process is structured around the primary goal of **creating a working local Method account**.

## Philosophy

Our onboarding approach is built on the principle that new developers should:
1. **Understand the system** - Know what Method is, how it works, and why it matters
2. **Develop troubleshooting skills** - Learn systematic approaches to problem-solving
3. **Work toward a clear goal** - Create a local account that demonstrates full functionality
4. **Build independence** - Become capable of solving problems and contributing effectively

## Documentation Structure

### Core Learning Documents

| Document | Purpose | When to Use |
|----------|---------|-------------|
| [Company Context](./company-context.md) | Understand Method's business model and value proposition | Day 1 - Before technical setup |
| [Architecture Overview](./architecture-overview.md) | Learn the technical architecture and system components | Day 1 - Foundation understanding |
| [Troubleshooting Skills](./troubleshooting-skills.md) | Develop systematic problem-solving approaches | Throughout - When issues arise |
| [First Week Schedule](./first-week-schedule.md) | Detailed daily plan with generous buffer time | Week 1 - Daily reference |
| [Learning Milestones](./learning-milestones.md) | 30/60/90 day progression and success criteria | Ongoing - Track progress |

### Process Documents

| Document | Purpose | When to Use |
|----------|---------|-------------|
| [Checklist](./checklist.md) | Track completion of setup and learning objectives | Daily - Progress tracking |
| [Buddy Assignment](./buddy-assignment.md) | Understand mentoring process and expectations | Day 1 - Meet your buddy |
| [Onboarding Manager Guide](./onboarding-manager-guide.md) | Complete guide for managers onboarding new team members | Pre-arrival through 90 days - Manager reference |
| [Onboarding Analysis](./onboarding-analysis.md) | Background on the onboarding system design | Reference - Understanding the approach |

## Quick Start Path

### For New Employees

#### Day 1: Foundation & Welcome
1. **9:00-9:30 AM**: Welcome meeting with your manager (motivation assessment, expectations)
2. **9:30-10:30 AM**: Read [Company Context](./company-context.md) and [Architecture Overview](./architecture-overview.md)
3. **10:30-11:00 AM**: Meet your assigned buddy
4. **Afternoon**: Begin [First Week Schedule](./first-week-schedule.md) Day 1 tasks
5. **End of Day**: Review [Learning Milestones](./learning-milestones.md) Week 1 goals

#### Week 1: Setup & Understanding
- Follow the [First Week Schedule](./first-week-schedule.md) daily plan
- **Day 2**: Attend team introductions meeting
- **Day 3**: Participate in Errol's product demo/bootcamp session
- Use [Troubleshooting Skills](./troubleshooting-skills.md) when issues arise
- Track progress with [Checklist](./checklist.md)
- Daily check-ins with buddy and manager

#### Month 1: Deep Dive & Contribution
- Follow [Learning Milestones](./learning-milestones.md) Days 8-30 plan
- Focus on understanding Method-specific patterns
- Make your first code contributions
- Develop advanced troubleshooting skills
- **30-day review** with manager

#### Months 2-3: Ownership & Leadership
- Follow [Learning Milestones](./learning-milestones.md) Days 31-90 plan
- Own features end-to-end
- Mentor other developers
- Drive process improvements
- **60-day and 90-day reviews** with manager

### For Managers

#### Pre-Arrival (1 Week Before)
- Review [Onboarding Manager Guide](./onboarding-manager-guide.md) pre-arrival checklist
- Assign buddy using [Buddy Assignment](./buddy-assignment.md) criteria
- Schedule Errol's product demo session for Day 3
- Set up first week calendar meetings
- Coordinate access provisioning with IT/Security

#### Day 1: Welcome & Foundation
- Conduct welcome meeting with motivation assessment
- Facilitate Slack introductions and channel setup
- Introduce new hire to assigned buddy
- Post team welcome message in #engineering

#### Week 1: Daily Support
- Daily 5-10 minute check-ins with new hire
- Weekly check-in with assigned buddy
- Monitor progress through [Checklist](./checklist.md)
- Address any escalated issues promptly

#### Ongoing: 30/60/90 Day Reviews
- Follow structured review agendas in [Onboarding Manager Guide](./onboarding-manager-guide.md)
- Track success metrics and intervene if needed
- Support career development planning
- Gather feedback to improve onboarding process

## Key Concepts

### What is Method?
Method is a no-code software platform with strong QuickBooks integration, built for small to medium businesses. We provide:
- Custom software solutions without traditional coding
- Deep QuickBooks integration for financial data
- CRM capabilities built on our platform
- Multi-tenant architecture with complete data isolation

### Technical Architecture
- **Frontend**: Method UI with legacy and modern app builders
- **Backend**: Gateway API, Runtime Core, and microservices
- **Data**: SQL Server (account data) + MongoDB (definitions)
- **Infrastructure**: IIS, RabbitMQ, Redis, Elasticsearch

### Troubleshooting Approach
- **5 Whys**: Get to root cause by asking "why" five times
- **Binary Search**: Systematically eliminate half the possibilities
- **Health Checks**: Use our comprehensive health check system
- **Log Analysis**: Know where to look for different types of problems

## Success Metrics

### Week 1 Goals
- [ ] Understand Method's business model and architecture
- [ ] Successfully set up local development environment
- [ ] Create and test your first local Method account
- [ ] Can troubleshoot common setup issues independently

### Month 1 Goals
- [ ] Submit first PR within 2 weeks
- [ ] Understand Method-specific development patterns
- [ ] Can debug issues in both frontend and backend code
- [ ] Contribute to troubleshooting documentation

### Month 2-3 Goals
- [ ] Own a feature from design to production
- [ ] Help other developers with their issues
- [ ] Contribute to process improvements
- [ ] Be recognized as a go-to person for complex problems

## Getting Help

### When to Ask for Help
- **Immediately**: Security issues, production problems, access problems
- **After 30 minutes**: Complex technical problems you can't solve
- **After 2 hours**: Any problem that's blocking your progress
- **Always**: When you're unsure about business logic or requirements

### How to Ask for Help
1. **Try first**: Document what you've tried
2. **Be specific**: Include error messages, steps to reproduce, environment details
3. **Show effort**: Demonstrate that you've attempted to solve it yourself
4. **Learn**: Ask questions to understand the solution, not just get it fixed

### Resources
- **Slack Channels**: #dev-help, #platform-team
- **Documentation**: This onboarding directory and main docs
- **Health Check**: `C:\MethodDev\DeveloperTools\Method-HealthChecks\publish\`
- **Logs**: `D:\logs\` directory
- **Build Commands**: `C:\MethodDev\DeveloperTools\DevSetup\build_local\projects.yaml`

## Learning Resources

### Books to Read
- **"Debugging" by David J. Agans**: Systematic approach to problem solving
- **"An Introduction to Thinking in Systems" by Donella Meadows**: Understanding complex systems
- **"Radical Candor" by Kim Scott**: Communication skills for getting help effectively

### Online Resources
- [Troubleshooting Decision Tree](../troubleshooting/decision-tree.md)
- [General Troubleshooting Guide](../troubleshooting/general.md)
- [Log Files Reference](../troubleshooting/log-files.md)
- [Health Check Application](../additional-tools/health-check.md)

## Remember

The goal isn't to never need help—it's to become the person others come to for help while knowing when and how to ask for it yourself. Every problem you solve makes you more valuable to the team and more confident in your abilities.

---

**Next Steps:**
1. Start with [Company Context](./company-context.md) to understand what Method is
2. Read [Architecture Overview](./architecture-overview.md) to understand how it works
3. Follow [First Week Schedule](./first-week-schedule.md) to get started
4. Use [Troubleshooting Skills](./troubleshooting-skills.md) when you encounter issues
5. Track your progress with [Learning Milestones](./learning-milestones.md)
