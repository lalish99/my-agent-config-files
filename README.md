# Cursor AI Instructions & Rules

A comprehensive collection of Cursor AI instructions and rules for building modern web applications with Next.js, TypeScript, and best practices.

## 🎯 What This Repository Contains

This repository contains carefully crafted Cursor AI instructions and rules that enforce architectural patterns, coding standards, and best practices for web development. These configurations help maintain consistency, quality, and developer experience across projects.

## 📁 Repository Structure

```
├── .cursor/
│   ├── instructions.md                    # Main Cursor AI instructions
│   └── rules/                            # Automated Cursor rules
│       ├── api-architecture.mdc          # API design patterns
│       ├── authentication-security.mdc   # Auth & security patterns
│       ├── component-organization.mdc    # Component structure
│       ├── form-validation.mdc           # Form handling patterns
│       ├── spanish-ui.mdc               # Internationalization rules
│       ├── typescript-standards.mdc     # TypeScript best practices
│       └── utility-organization.mdc     # Utility organization
├── .github/
│   └── instructions/                     # Detailed architectural guides
│       ├── development-standards.instructions.md
│       ├── security-instructions.instructions.md
│       └── server-client-architecture.instructions.md
└── README.md                            # This file
```

## 🏗️ Key Architectural Principles

### **API-First Design**
- **API routes** for all mutations (POST, PATCH, DELETE)
- **No server actions** - ensures mobile app compatibility
- **Direct Prisma calls** in server components for data fetching

### **Type Safety**
- **Direct Prisma types** - no custom type abstractions
- **Strict TypeScript** - no `any` types allowed
- **Proper imports** - type-only imports when appropriate

### **Internationalization**
- **Spanish UI text** - all user-facing content in Spanish
- **English code** - variables, functions, comments in English
- **Centralized error messages** - consistent error handling

### **Security & Authentication**
- **Data scoping** - always filter by authenticated user
- **Role-based access** - proper authorization patterns
- **Secure patterns** - following security best practices

## 🚀 How to Use

### **For New Projects**
1. Copy the `.cursor/` directory to your project root
2. Copy the `.github/instructions/` directory for detailed guides
3. Configure your project to follow the established patterns

### **For Existing Projects**
1. Review the architectural principles
2. Gradually adopt the patterns that fit your needs
3. Use the rules as guidelines for refactoring

## 📚 Documentation

### **Main Instructions**
- **[`.cursor/instructions.md`](.cursor/instructions.md)** - Comprehensive development guide with examples and patterns

### **Detailed Guides**
- **[Server-Client Architecture](.github/instructions/server-client-architecture.instructions.md)** - Component separation patterns
- **[Development Standards](.github/instructions/development-standards.instructions.md)** - Coding standards and preferences
- **[Security Instructions](.github/instructions/security-instructions.instructions.md)** - Security guidelines and patterns

### **Automated Rules**
- **[Rules Overview](.cursor/rules/README.md)** - Complete guide to all automated Cursor rules

## 🎨 Example Patterns

### **API Route Structure**
```typescript
export async function POST(request: NextRequest) {
  const { userId } = await auth();
  if (!userId) {
    return NextResponse.json({ error: ERROR_MESSAGES.UNAUTHORIZED }, { status: 401 });
  }

  const validatedData = Schema.parse(await request.json());
  const result = await prisma.model.create({
    data: { ...validatedData, userId }
  });

  return NextResponse.json({ result }, { status: 201 });
}
```

### **Component Architecture**
```typescript
// Server Component (data fetching)
export async function DataSC() {
  const data = await prisma.model.findMany();
  return <ClientComponent data={data} />;
}

// Client Component (interactions)
"use client";
export function ClientComponent({ data }) {
  const handleSubmit = async (formData) => {
    await fetch('/api/endpoint', { method: 'POST', body: JSON.stringify(formData) });
    router.refresh();
  };
  return <form onSubmit={handleSubmit}>...</form>;
}
```

### **Internationalization**
```typescript
// Centralized error messages
const ERROR_MESSAGES = {
  PATIENT_NOT_FOUND: "Paciente no encontrado",
  VALIDATION_FAILED: "Validación fallida"
};

// Spanish UI with English code
<TextField
  label="Nombre Completo"           // Spanish label
  placeholder="Ingresa el nombre"   // Spanish placeholder
  error={!!validationErrors.fullName}  // English code
/>
```

## 🔧 Technology Stack

These instructions are designed for modern web applications using:

- **Framework**: Next.js 14+ with App Router
- **Language**: TypeScript (strict mode)
- **Database**: Prisma ORM with PostgreSQL
- **Authentication**: Clerk
- **UI Framework**: Material-UI (MUI)
- **Validation**: Zod
- **Styling**: MUI theming system

## 📋 Quality Standards

### **Code Quality**
- Files under 350 lines
- Comprehensive error handling
- Loading and error states
- Client-side validation
- TypeScript compilation passes

### **Architecture**
- Server-client component separation
- API routes for mutations
- Direct Prisma type usage
- Proper data scoping
- Role-based access control

### **User Experience**
- Spanish UI text
- Centralized error messages
- Loading states with Suspense
- Form validation feedback
- Responsive design

## 🤝 Contributing

This repository represents a collection of best practices and patterns. Feel free to:

1. **Fork** the repository for your own use
2. **Adapt** the patterns to fit your specific needs
3. **Share** improvements or additional patterns
4. **Use** as a reference for architectural decisions

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

## 🙏 Acknowledgments

These instructions and rules are based on real-world development experience and best practices from the web development community. They represent patterns that have proven effective for building maintainable, scalable applications.

---

**Note**: These instructions are designed to be technology-agnostic where possible, but some patterns are specific to the Next.js/React ecosystem. Adapt them to your preferred technology stack as needed.
