---
name: marketplace-code-generator
description: Generate marketplace-related code. Use when user mentions products, orders, payments, shopping cart, e-commerce, or marketplace features.
---

# Marketplace Code Generator

This skill helps generate high-quality marketplace application code.

## Capabilities

- Generate TypeScript interfaces for products, orders, users
- Create API endpoint templates (REST/GraphQL)
- Build React/Next.js components for storefronts
- Design database schemas for e-commerce
- Implement shopping cart logic
- Set up payment integration stubs

## Code Standards

- Always use TypeScript with strict typing
- Follow React/Next.js best practices
- Implement proper error handling with try-catch
- Use environment variables for secrets (never hardcode)
- Write unit tests for business logic
- Use Zod for input validation

## File Organization

- Keep components under 400 lines
- Extract utilities from large components
- Organize by feature/domain, not by type

## Output Format

When generating code, include:
1. Type definitions
2. Main implementation
3. Error handling
4. Usage examples

## Activation

This skill activates when you need help with:
- Product management
- Order processing
- Payment integration
- Shopping cart implementation
- User authentication for e-commerce
- Inventory management
- Shopping checkout flow
