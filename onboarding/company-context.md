# Method: Company Context & Business Model

## What is Method?

Method is a **no-code software platform** with strong QuickBooks integration, built specifically for small to medium businesses. We provide:

- **Custom Software Solutions**: Extremely customizable software tailored to each business's unique needs
- **No-Code Development**: Users build screens and define business logic without traditional coding
- **QuickBooks Integration**: Deep integration with QuickBooks for financial data synchronization
- **CRM Capabilities**: Customer relationship management built on top of our platform

## Our Business Model

### Target Market
- **Small to Medium Businesses (SMBs)**: Companies that need custom software but can't afford traditional development
- **QuickBooks Users**: Businesses already using QuickBooks for financial management
- **Custom Software Needs**: Companies with unique processes that off-the-shelf software can't handle

### Value Proposition
1. **Cost-Effective Custom Software**: Get custom software at a fraction of traditional development costs
2. **QuickBooks Integration**: Seamless data flow between financial and operational systems
3. **No-Code Flexibility**: Business users can modify and extend their software without developers
4. **Rapid Deployment**: Get custom software running in weeks, not months

### Revenue Model
- **Subscription-Based**: Monthly/annual fees per account
- **Tiered Pricing**: Different feature sets for different business sizes
- **App Store**: Additional revenue from specialized apps and integrations

## Technical Architecture Overview

### Platform Components

#### Frontend Layer
- **Method UI**: Main application interface
- **Legacy Screen Builder**: Original no-code interface
- **Modern App Builder**: Newer, more powerful interface
- **Action Editor**: Visual business logic definition

#### Backend Layer
- **Gateway API**: Main API gateway and routing
- **Runtime Core**: Business logic execution engine
- **MS Gateway**: Microservice gateway for external integrations
- **Tables & Fields**: Database schema management

#### Data Layer
- **SQL Server**: Account-specific databases with:
  - Spider tables (user-defined tables)
  - Account tables (QuickBooks sync)
  - Custom tables (business-specific data)
- **MongoDB**: Screen definitions, app configurations, user preferences
- **QuickBooks API**: Financial data synchronization

#### Infrastructure
- **IIS**: Web server hosting
- **RabbitMQ**: Message queue for app installations
- **Redis**: Caching layer
- **Elasticsearch**: Search functionality

### Multi-Tenant Architecture

Each customer gets:
- **Isolated SQL Database**: Complete data separation
- **Isolated MongoDB**: Screen and configuration separation
- **Shared Infrastructure**: Cost-effective hosting
- **Customizable Experience**: Per-account and per-user customization

## App Development & Distribution

### Development Process
1. **Build in TemplateDevV2**: Development database for creating new apps
2. **Test & Validate**: Ensure app works correctly
3. **Publish to MethodAppStore**: Make app available to customers
4. **Customer Installation**: App gets copied to customer's account

### App Types
- **Stock Apps**: Pre-built apps available to all customers
- **Custom Apps**: Customer-specific applications
- **Third-Party Apps**: Apps built by external developers

### App Components
- **Screens**: User interface definitions
- **Actions**: Business logic components
- **App Routines**: Reusable logic across screens
- **Data Models**: Database schema definitions

## Key Business Concepts

### Accounts
- **Definition**: A company or organization using Method
- **Isolation**: Each account has completely separate data and configuration
- **Customization**: Accounts can customize screens, workflows, and data models

### Users
- **Definition**: People who have access to an account
- **Roles**: Different permission levels (admin, user, customizer)
- **Customization**: Users can personalize their experience

### Screens
- **Definition**: User interface pages built with our no-code tools
- **Types**: Runtime screens (what users see) vs. design screens (what builders use)
- **Customization**: Can be customized at account and user levels

### Actions
- **Definition**: Units of business logic (Send Email, Update Record, etc.)
- **Composition**: Actions can be grouped into Action Sets
- **Reusability**: Actions can be reused across different screens

### Tables & Fields
- **Spider Tables**: User-defined data structures
- **Account Tables**: QuickBooks-synced data
- **Custom Tables**: Business-specific data not in QuickBooks
- **Fields**: Individual data attributes within tables

## Development Philosophy

### No-Code Approach
- **Visual Development**: Drag-and-drop interface building
- **Business Logic**: Defined through actions, not code
- **User Empowerment**: Business users can modify their own software
- **Rapid Iteration**: Changes can be made and deployed quickly

### Event-Driven Architecture
- **User Events**: Clicks, form submissions, navigation
- **System Events**: Data changes, scheduled tasks, external triggers
- **Action Execution**: Business logic runs in response to events
- **Real-time Updates**: Changes propagate immediately

### Multi-Tenant Design
- **Data Isolation**: Complete separation between customers
- **Shared Resources**: Cost-effective infrastructure
- **Customization**: Per-account and per-user personalization
- **Scalability**: Easy to add new customers

## Why This Matters for Developers

### Understanding the Business
- **Customer Needs**: Why we build what we build
- **Technical Decisions**: How business requirements drive architecture
- **User Experience**: How our technical choices affect end users
- **Scalability**: How our architecture supports business growth

### Development Context
- **Multi-Tenant Awareness**: Every change affects multiple customers
- **Data Sensitivity**: Customer data must be completely isolated
- **Performance**: System must scale with customer growth
- **Reliability**: Downtime affects multiple customers

### Problem-Solving Context
- **Business Impact**: How technical issues affect customers
- **Root Cause Analysis**: Understanding why problems occur
- **Prevention**: Building systems that prevent future issues
- **Documentation**: Helping others understand and maintain the system

## Success Metrics

### Business Metrics
- **Customer Growth**: Number of new accounts
- **Revenue Growth**: Monthly recurring revenue
- **Customer Satisfaction**: Net Promoter Score
- **Churn Rate**: Customer retention

### Technical Metrics
- **System Uptime**: 99.9% availability target
- **Performance**: Response time targets
- **Scalability**: Ability to handle growth
- **Security**: Data protection and compliance

### Developer Metrics
- **Time to First PR**: How quickly new developers contribute
- **Bug Resolution**: How quickly issues are fixed
- **Code Quality**: Maintainability and reliability
- **Knowledge Sharing**: Documentation and mentoring

## Learning Path for New Developers

### Week 1: Foundation
- Understand what Method does and why
- Learn the technical architecture
- Set up local development environment
- Create your first local account

### Month 1: Deep Dive
- Understand the codebase structure
- Learn Method-specific patterns
- Make your first code contributions
- Understand the development workflow

### Month 2-3: Ownership
- Own features end-to-end
- Mentor other developers
- Contribute to process improvements
- Become a go-to person for complex problems

## Resources for Further Learning

### Business Understanding
- [Company Terms](../Company%20Terms.md) - Platform terminology
- [Architecture Overview](./architecture-overview.md) - Technical architecture
- Customer demos and case studies
- Sales and marketing materials

### Technical Understanding
- [Troubleshooting Skills](./troubleshooting-skills.md) - Problem-solving approach
- [Learning Milestones](./learning-milestones.md) - Structured learning plan
- Codebase exploration and documentation
- Pair programming with experienced developers

### Process Understanding
- [First Week Schedule](./first-week-schedule.md) - Getting started
- [Buddy Assignment](./buddy-assignment.md) - Mentoring process
- Code review and deployment processes
- Incident response and on-call procedures

Remember: Understanding the business context makes you a more effective developer. You'll make better technical decisions, write better code, and solve problems more effectively when you understand why things work the way they do.

---

**Related Documentation:**
- [Architecture Overview](./architecture-overview.md)
- [Company Terms](../Company%20Terms.md)
- [Learning Milestones](./learning-milestones.md)
- [Troubleshooting Skills](./troubleshooting-skills.md)
