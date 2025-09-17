# First Week Schedule: From Zero to Local Account

## Overview

This schedule is designed to take you from your first day to having a working local Method account. We've allocated **generous time** for setup because we know there are almost always complications with .NET Core versions, Windows permissions, IIS configs, database backups, or other environmental issues.

**Goal**: Create your first local Method account by the end of the week
**Philosophy**: Learn by doing, with systematic troubleshooting when things go wrong

## Day 1: Foundation & Understanding (8 hours)

### Morning (4 hours): System Understanding
- **9:00-9:30**: Welcome meeting with your buddy
- **9:30-10:30**: Read [Architecture Overview](./architecture-overview.md)
- **10:30-11:00**: Review [Company Terms](../Company%20Terms.md)
- **11:00-12:00**: Walkthrough of existing Method account with buddy

### Afternoon (4 hours): Prerequisites Setup
- **1:00-2:00**: Complete [Prerequisites](../prerequisites/README.md)
- **2:00-3:00**: Complete [Credentials](../credentials/README.md)
- **3:00-4:00**: Complete [GitHub Setup](../github-setup/README.md)
- **4:00-5:00**: **Buffer time** for any credential/access issues

**End of Day Goal**: Understand what Method is and have basic access credentials

## Day 2: Development Environment (8 hours)

### Morning (4 hours): Software Installation
- **9:00-10:00**: Complete [Software Installation](../software-installation/README.md)
- **10:00-11:00**: Complete [Microsoft Tools](../microsoft-tools/README.md)
- **11:00-12:00**: **Buffer time** for installation issues

### Afternoon (4 hours): Environment Configuration
- **1:00-2:00**: Complete [Environment Setup](../environment-setup/README.md)
- **2:00-3:00**: Complete [Certificates](../certificates/README.md)
- **3:00-4:00**: Complete [IIS Configuration](../iis-configuration/README.md)
- **4:00-5:00**: **Buffer time** for configuration issues

**End of Day Goal**: Have all required software installed and configured

## Day 3: Database & Core Services (8 hours)

### Morning (4 hours): Database Setup
- **9:00-10:30**: Complete [SQL Server Setup](../software-installation/sql-server.md)
- **10:30-11:30**: Complete [Database Restore](../troubleshooting/database-restore.md)
- **11:30-12:00**: **Buffer time** for database issues

### Afternoon (4 hours): .NET Core & Services
- **1:00-2:00**: Complete [.NET Core Requirements](../netcore-requirements/README.md)
- **2:00-3:00**: Start Docker services (MongoDB, Redis, Elasticsearch, RabbitMQ)
- **3:00-4:00**: **Buffer time** for service startup issues
- **4:00-5:00**: First health check run

**End of Day Goal**: Have all databases and core services running

## Day 4: Local Development Setup (8 hours)

### Morning (4 hours): Build & Test
- **9:00-10:00**: Complete [Local Development Prerequisites](../local-development/prerequisites.md)
- **10:00-11:00**: Build critical projects
- **11:00-12:00**: **Buffer time** for build issues

### Afternoon (4 hours): Health Check & Troubleshooting
- **1:00-2:00**: Run comprehensive health check
- **2:00-3:00**: Address any health check failures
- **3:00-4:00**: **Buffer time** for troubleshooting
- **4:00-5:00**: Practice with [Troubleshooting Skills](./troubleshooting-skills.md)

**End of Day Goal**: Health check shows all green

## Day 5: Create Your First Account (8 hours)

### Morning (4 hours): Account Creation
- **9:00-10:00**: Follow [Create Local Account](../local-development/create-account.md)
- **10:00-11:00**: **Buffer time** for account creation issues
- **11:00-12:00**: Test basic functionality

### Afternoon (4 hours): Validation & Learning
- **1:00-2:00**: Validate account works (login, basic screens)
- **2:00-3:00**: Explore the platform with your buddy
- **3:00-4:00**: **Buffer time** for any final issues
- **4:00-5:00**: Document what you learned and any issues encountered

**End of Day Goal**: Successfully created and tested your first local Method account

## Daily Check-in Structure

### Morning Standup (15 minutes)
- What did you accomplish yesterday?
- What are you working on today?
- Any blockers or questions?

### Buddy Check-in (30 minutes)
- Review progress from previous day
- Discuss any issues encountered
- Plan for current day's work
- Practice troubleshooting techniques

### End-of-Day Review (15 minutes)
- What went well?
- What was challenging?
- What would you do differently?
- Questions for tomorrow

## Buffer Time Philosophy

We've allocated **2-3 hours of buffer time per day** because:

1. **Environmental Issues**: .NET Core version conflicts, Windows permissions, IIS configuration problems
2. **Database Issues**: Backup restoration failures, permission problems, connection string issues
3. **Learning Curve**: Understanding new concepts takes time
4. **Troubleshooting Practice**: Learning to solve problems is part of the process
5. **Realistic Expectations**: Better to finish early than feel rushed

## Success Criteria

### End of Day 1
- [ ] Understand Method's business model and architecture
- [ ] Have all required credentials and access
- [ ] Can explain what you're building toward

### End of Day 2
- [ ] All required software installed
- [ ] Development environment configured
- [ ] Can explain the purpose of each tool

### End of Day 3
- [ ] All databases running and accessible
- [ ] Core services (Docker containers) operational
- [ ] Can connect to databases via SSMS

### End of Day 4
- [ ] Health check shows all green
- [ ] Can build and run Method projects
- [ ] Understand troubleshooting methodology

### End of Day 5
- [ ] Successfully created local Method account
- [ ] Can log in and navigate the platform
- [ ] Understand the relationship between SQL and MongoDB data
- [ ] Can explain the app installation process

## When Things Go Wrong

### Common Issues and Time Estimates

| Issue Type | Typical Time to Resolve | Escalation Point |
|------------|------------------------|------------------|
| .NET Core version conflicts | 1-2 hours | After 2 hours |
| IIS configuration problems | 1-3 hours | After 3 hours |
| Database permission issues | 2-4 hours | After 4 hours |
| Docker container failures | 30 minutes - 2 hours | After 2 hours |
| Network/VPN connectivity | 30 minutes - 1 hour | After 1 hour |

### Escalation Process

1. **Try documented solutions** (15-30 minutes)
2. **Ask your buddy** (they've seen these issues before)
3. **Post in #dev-help** with specific error details
4. **Escalate to platform team** if it's a system-wide issue

### Learning from Failures

Every problem you solve makes you more valuable:
- **Document the solution** for future reference
- **Update our troubleshooting guides** if you find gaps
- **Share your learnings** with other new developers
- **Celebrate the win** - you just leveled up!

## Week 1 Success Metrics

By the end of your first week, you should be able to:

1. **Explain Method's architecture** to a new developer
2. **Troubleshoot common issues** using our decision tree
3. **Create and manage local accounts** independently
4. **Navigate the codebase** to find relevant files
5. **Use our health check system** effectively
6. **Ask for help effectively** when needed

## Preparation for Week 2

At the end of Week 1, you'll be ready to:
- Start working on your first PR
- Understand the code review process
- Begin learning specific Method development patterns
- Contribute to our documentation improvements

Remember: The goal isn't just to get your environment working—it's to develop the skills and confidence to solve problems independently while knowing when and how to ask for help.

---

**Related Documentation:**
- [Architecture Overview](./architecture-overview.md)
- [Troubleshooting Skills](./troubleshooting-skills.md)
- [Learning Milestones](./learning-milestones.md)
- [Buddy Assignment](./buddy-assignment.md)
