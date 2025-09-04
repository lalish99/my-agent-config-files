# Cursor Rules - Nuvira Medical App

## 📋 Overview

This directory contains Cursor rules that encode Eduardo's development standards and architectural decisions for the Nuvira medical application. These rules will automatically guide Cursor AI to follow established patterns and best practices.

## 🔧 Generated Rules

### **Always Applied Rules** (Applied to every request)

| Rule | File | Description |
|------|------|-------------|
| **API Architecture** | `api-architecture.mdc` | Enforces API routes over server actions, mobile-first design |
| **Spanish UI Standards** | `spanish-ui.mdc` | Spanish UI text with English code, centralized error messages |

### **TypeScript-Specific Rules** (Applied to `.ts` and `.tsx` files)

| Rule | File | Description |
|------|------|-------------|
| **TypeScript Standards** | `typescript-standards.mdc` | Direct Prisma types, import standards, file size limits |
| **Component Organization** | `component-organization.mdc` | Server-client separation, file structure patterns |

### **Context-Aware Rules** (Applied when relevant)

| Rule | File | Description | Trigger |
|------|------|-------------|---------|
| **Authentication Security** | `authentication-security.mdc` | Clerk integration, data scoping patterns | User authentication work |
| **Form Validation** | `form-validation.mdc` | Client-side validation, error handling | Form development |

### **Utility-Specific Rules** (Applied to `lib/**/*.ts` files)

| Rule | File | Description |
|------|------|-------------|
| **Utility Organization** | `utility-organization.mdc` | Centralized utilities, error messages, date formatting |

## 🎯 Key Architectural Principles Enforced

### 1. **Mobile-First API Design**
- ✅ API routes for all mutations
- ❌ Server actions prohibited
- ✅ Direct Prisma calls in server components

### 2. **Type Safety**
- ✅ Direct Prisma type usage
- ❌ Custom type abstractions
- ✅ Strict TypeScript with no `any`

### 3. **Internationalization**
- ✅ Spanish UI text with centralized error messages
- ✅ English code (variables, functions, comments)
- ✅ Consistent error handling

### 4. **Security & Authentication**
- ✅ Data scoped to authenticated users
- ✅ Proper Clerk integration patterns
- ✅ Role-based access control

### 5. **Component Architecture**
- ✅ Server-client component separation
- ✅ Required loading/error states
- ✅ Organized file structure

### 6. **Code Quality**
- ✅ 350-line file limit
- ✅ Client-side form validation
- ✅ Centralized utility functions

## 📚 Related Documentation

For detailed architectural guidelines, see:
- [Server-Client Architecture](.github/instructions/server-client-architecture.instructions.md)
- [Development Standards](.github/instructions/development-standards.instructions.md)
- [Main Cursor Instructions](.cursor/instructions.md)

## 🔄 How Rules Work

1. **Always Applied**: Rules with `alwaysApply: true` are included in every Cursor request
2. **File-Specific**: Rules with `globs` patterns apply only to matching files
3. **Context-Aware**: Rules with `description` are applied when Cursor determines they're relevant

## ✅ Success Criteria

When these rules are active, Cursor will automatically:
- Suggest API routes instead of server actions
- Use direct Prisma types instead of custom abstractions
- Include Spanish UI text with centralized error messages
- Implement proper authentication and data scoping
- Follow established component organization patterns
- Enforce form validation and error handling standards

---

**Generated**: July 30, 2025  
**Based on**: Eduardo's development standards and project patterns  
**Purpose**: Ensure consistent, high-quality code generation aligned with architectural decisions