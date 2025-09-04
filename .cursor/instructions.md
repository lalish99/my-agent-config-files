# Cursor AI Instructions - Nuvira Medical App

## 🎯 Project Overview

Nuvira is a medical application built with Next.js App Router, TypeScript, Prisma, Clerk authentication, and MUI. The app serves both doctors and patients with role-based access and specialized workflows.

## 📚 Complete Instructions Reference

**This file provides a comprehensive overview. For detailed architectural guidelines, also refer to:**
- `.github/instructions/server-client-architecture.instructions.md` - Server-client component patterns
- `.github/instructions/development-standards.instructions.md` - Eduardo's coding standards and preferences
- `.github/instructions/security-instructions.instructions.md` - Security guidelines
- `.cursor/rules/` - Automated Cursor rules that enforce these standards

## 🏗️ Architecture Standards

### **API Design Philosophy**
- **CRITICAL**: Use API routes (POST, PATCH, DELETE) for all mutations - NEVER server actions
- **Reason**: Mobile app compatibility is a key future requirement
- **Data Fetching**: Server components use direct Prisma calls only
- **Pattern**: Server Component (initial data) → API routes (mutations) → client refresh

### **Server-Client Component Separation**
```typescript
// ✅ CORRECT: Server Component for data fetching
export async function DataSC() {
  const data = await prisma.model.findMany(); // Direct Prisma call
  return <ClientComponent data={data} />;
}

// ✅ CORRECT: Client Component for interactions
"use client";
export function ClientComponent({ data }) {
  const handleSubmit = async (formData) => {
    await fetch('/api/endpoint', { method: 'POST', body: JSON.stringify(formData) });
    router.refresh(); // Revalidate server component data
  };
  return <form onSubmit={handleSubmit}>...</form>;
}

// ❌ WRONG: Server actions
async function serverAction() {
  // Don't use server actions for mutations
}

// ❌ WRONG: Fetch in server component
export async function BadSC() {
  const data = await fetch('/api/endpoint'); // Use direct Prisma instead
}
```

### **Type System Standards**
- **Use Prisma Types Directly**: Import from `@prisma/client`, avoid custom type abstractions
- **No Type Wrappers**: Don't create custom types that just wrap Prisma types
- **Type Safety**: Strict TypeScript with proper imports

```typescript
// ✅ CORRECT: Direct Prisma type usage
import type { DoctorPatient } from "@prisma/client";
interface Props { patients: DoctorPatient[]; }

// ❌ WRONG: Custom type abstraction
export type PatientTableData = DoctorPatient; // Unnecessary layer
```

## 🌐 Internationalization Rules

### **Spanish UI, English Code**
- **All User-Facing Text**: Labels, buttons, error messages, placeholders in Spanish
- **All Code**: Variable names, function names, comments in English
- **Error Messages**: Always use centralized `ERROR_MESSAGES` from `lib/utils/errorMessages.ts`

```typescript
// ✅ CORRECT: Spanish UI text with English code
const ERROR_MESSAGES = {
  PATIENT_NOT_FOUND: "Paciente no encontrado",
  VALIDATION_FAILED: "Validación fallida"
};

<TextField 
  label="Nombre Completo"           // Spanish label
  placeholder="Ingresa el nombre"   // Spanish placeholder
  error={!!validationErrors.fullName}  // English code
/>

// ❌ WRONG: Hardcoded Spanish strings
{ error: "Un paciente con esta información ya existe." } // Use ERROR_MESSAGES instead
```

## 🔐 Authentication & Authorization

### **Clerk Integration Patterns**
- **Role-Based Access**: Doctor vs Patient with different capabilities
- **Data Scoping**: Always scope data to authenticated user
- **Business Rules**: 
  - Doctors only see their own `DoctorPatient` records
  - Doctors CANNOT access registered `Patient` profiles
  - Patients only see their own data

```typescript
// ✅ CORRECT: Proper doctor data scoping
const { userId } = await auth();
const doctorProfile = await prisma.doctorProfile.findUnique({
  where: { userId }
});

const patients = await prisma.doctorPatient.findMany({
  where: { doctorId: doctorProfile.id } // Always scope to doctor
});

// ❌ WRONG: Unscoped data access
const patients = await prisma.doctorPatient.findMany(); // Missing doctor filter
```

## 📁 File Organization Standards

### **Component Structure**
```
app/[role]/[feature]/
├── components/           # Feature-specific components
│   ├── FeatureSC.tsx    # Server component (data fetching)
│   ├── FeatureTable.tsx # Client component (UI)
│   └── FeatureModal.tsx # Client component (interactions)
├── [id]/                # Dynamic routes
│   ├── page.tsx         # Route page
│   ├── loading.tsx      # Loading state (required)
│   └── error.tsx        # Error state (required)
├── page.tsx             # Main feature page
├── loading.tsx          # Feature loading state
└── error.tsx           # Feature error state
```

### **Utility Organization**
```
lib/
├── utils/
│   ├── errorMessages.ts  # Centralized Spanish error messages
│   ├── dateUtils.ts      # Date formatting utilities
│   └── validation.ts     # Shared validation logic
├── prisma.ts            # Prisma client
└── auth-config.ts       # Clerk configuration
```

## 🎨 UI/UX Standards

### **Material-UI Integration**
- **Component Library**: Use MUI for all UI components
- **Loading States**: Implement Skeleton components for better UX
- **Form Validation**: Client-side validation before API submission
- **Error Handling**: Graceful error boundaries with Spanish messages

### **Required UI Patterns**
```typescript
// ✅ Loading states with Suspense
<Suspense fallback={<Loading />}>
  <DataSC />
</Suspense>

// ✅ Client-side form validation
const validateForm = () => {
  const errors: Record<string, string> = {};
  if (!form.fullName.trim()) {
    errors.fullName = ERROR_MESSAGES.FULL_NAME_REQUIRED;
  }
  return Object.keys(errors).length === 0;
};

// ✅ Spanish UI text
<Typography variant="h4">Gestión de Pacientes</Typography>
<Button variant="contained">Agregar Paciente</Button>
```

## 🔧 Code Quality Standards

### **File Size Limits**
- **Components**: Keep under 350 lines
- **Extract Logic**: Move complex logic to utilities
- **Split Components**: Break large components into smaller ones

### **TypeScript Requirements**
- **Strict Mode**: No `any` types allowed
- **Proper Imports**: Use type-only imports when appropriate
- **Interface Definitions**: Clear, descriptive interfaces

### **Validation Patterns**
```typescript
// ✅ CORRECT: Zod validation in API routes
const Schema = z.object({
  fullName: z.string().min(1, ERROR_MESSAGES.FULL_NAME_REQUIRED),
  email: z.string().email().optional().or(z.literal(""))
});

// ✅ CORRECT: Client-side validation before submission
const handleSubmit = (formData) => {
  if (!validateForm()) return; // Validate first
  
  startTransition(async () => {
    const response = await fetch('/api/endpoint', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });
    
    if (response.ok) {
      router.refresh(); // Revalidate data
    } else {
      setError(ERROR_MESSAGES.UNEXPECTED_ERROR);
    }
  });
};
```

## 📚 Documentation Requirements

### **For Major Changes**
Create comprehensive documentation in `.docs/YYYYMMDD/feature-name/`:
- `README.md` - Main overview and architecture
- `technical-changes.md` - Detailed code analysis
- `quick-reference.md` - Developer cheat sheet
- `index.md` - Navigation and summary

**Note**: Use descriptive feature names (e.g., `patient-management`, `appointment-scheduling`, `notes-system`) to avoid collisions when merging branches.

### **Commit Message Standards**
- **Searchable Keywords**: Include relevant terms for AI discoverability
- **Structured Format**: Clear sections with technical details
- **Before/After Examples**: Show code changes
- **Migration Notes**: Guidance for similar future work

## 🚀 Development Workflow

### **Quality Gates**
1. **TypeScript Compilation**: Must pass `tsc --noEmit`
2. **Linting**: No ESLint errors
3. **Pre-commit Hooks**: Automated formatting and checks
4. **Manual Testing**: Verify functionality works as expected

### **Testing Checklist**
- [ ] TypeScript compilation passes
- [ ] No linting errors
- [ ] All forms validate correctly
- [ ] API endpoints respond properly
- [ ] UI text displays in Spanish
- [ ] Loading/error states work
- [ ] Authentication/authorization enforced

## 🔍 Common Patterns & Examples

### **API Route Structure**
```typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@clerk/nextjs/server";
import { prisma } from "@/lib/prisma";
import { z } from "zod";
import { ERROR_MESSAGES } from "@/lib/utils/errorMessages";

export async function POST(request: NextRequest) {
  try {
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json({ error: ERROR_MESSAGES.UNAUTHORIZED }, { status: 401 });
    }

    const body = await request.json();
    const validatedData = Schema.parse(body);
    
    // Business logic with proper scoping
    const result = await prisma.model.create({
      data: { ...validatedData, userId }
    });

    return NextResponse.json({ result }, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: ERROR_MESSAGES.VALIDATION_FAILED, details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: ERROR_MESSAGES.UNEXPECTED_ERROR },
      { status: 500 }
    );
  }
}
```

### **Page Component Structure**
```typescript
import { Suspense } from "react";
import { DataSC } from "./components/DataSC";
import Loading from "./loading";

export default function FeaturePage() {
  return (
    <Suspense fallback={<Loading />}>
      <DataSC />
    </Suspense>
  );
}
```

## ❌ Common Anti-Patterns to Avoid

1. **Server Actions for Mutations** - Use API routes instead
2. **Custom Type Abstractions** - Use Prisma types directly
3. **Hardcoded Spanish Strings** - Use ERROR_MESSAGES
4. **Fetch in Server Components** - Use direct Prisma calls
5. **Missing Loading/Error States** - Always implement both
6. **Unscoped Data Access** - Always filter by user/role
7. **Files Over 350 Lines** - Extract components/utilities
8. **English UI Text** - All user-facing text in Spanish

## 🎯 Success Criteria

When implementing features, ensure:
- ✅ API routes for all mutations
- ✅ Direct Prisma types usage
- ✅ Spanish UI with centralized error messages
- ✅ Proper authentication and data scoping
- ✅ Loading and error states implemented
- ✅ Client-side validation before submission
- ✅ TypeScript compilation passes
- ✅ Comprehensive documentation for major changes

## 📞 Key Contacts & Resources

- **Project Owner**: Eduardo (prefers quality over speed)
- **Architecture**: Next.js App Router + Prisma + Clerk + MUI
- **Database**: PostgreSQL via Prisma ORM
- **Authentication**: Clerk with role-based access
- **UI Framework**: Material-UI (MUI)
- **Validation**: Zod for API routes, custom validation for client

---

**Remember**: Eduardo values maintainability, type safety, and comprehensive documentation. When in doubt, prioritize code quality and developer experience over quick implementations.