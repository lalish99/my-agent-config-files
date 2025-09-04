---
applyTo: '**'
---

# Eduardo's Development Standards & Preferences

## 🎯 Critical Architectural Decisions

### API Design Philosophy
- **MANDATORY**: Use API routes (POST, PATCH, DELETE) for ALL mutations - NEVER server actions
- **Reasoning**: Mobile app compatibility is a key future requirement
- **Data Fetching**: Server components use direct Prisma calls ONLY
- **Pattern**: Server Component (initial data) → API routes (mutations) → client refresh via `router.refresh()`

### Server Actions Policy
```typescript
// ❌ PROHIBITED: Server actions for mutations
"use server";
async function createPatient(formData) {
  // This pattern is NOT allowed
}

// ✅ REQUIRED: API routes for mutations
export async function POST(request: NextRequest) {
  // All mutations must go through API routes
  const result = await prisma.model.create(data);
  return NextResponse.json({ result });
}
```

## 🌐 Internationalization Standards

### Spanish UI, English Code
- **All User-Facing Text**: Labels, buttons, error messages, placeholders in Spanish
- **All Code Elements**: Variable names, function names, comments, types in English
- **Error Messages**: ALWAYS use centralized `ERROR_MESSAGES` from `lib/utils/errorMessages.ts`
- **NO Hardcoded Strings**: All Spanish text must be centralized

```typescript
// ✅ CORRECT: Centralized Spanish error messages
export const ERROR_MESSAGES = {
  PATIENT_NOT_FOUND: "Paciente no encontrado",
  VALIDATION_FAILED: "Validación fallida",
  UNAUTHORIZED: "No autorizado"
};

// ✅ CORRECT: Spanish UI with English code
<TextField 
  label="Nombre Completo"                    // Spanish UI
  error={!!validationErrors.fullName}       // English code
  helperText={ERROR_MESSAGES.NAME_REQUIRED} // Centralized Spanish
/>

// ❌ WRONG: Hardcoded Spanish strings
{ error: "Un paciente con esta información ya existe." }
```

## 🏗️ Type System Standards

### Direct Prisma Type Usage
- **Use Prisma Types Directly**: Import from `@prisma/client`, NO custom type abstractions
- **No Type Wrappers**: Avoid creating custom types that just duplicate Prisma types
- **Type Safety**: Strict TypeScript with proper imports and interfaces

```typescript
// ✅ CORRECT: Direct Prisma type usage
import type { DoctorPatient } from "@prisma/client";
interface PatientTableProps {
  patients: DoctorPatient[];
}

// ❌ WRONG: Unnecessary custom type abstraction
export type PatientTableData = DoctorPatient; // Don't create this
import { PatientTableData } from "./types";   // Don't do this
```

### Type Import Standards
```typescript
// ✅ CORRECT: Type-only imports
import type { DoctorPatient, Appointment } from "@prisma/client";
import { NextRequest, NextResponse } from "next/server";

// ✅ CORRECT: Regular imports for runtime values
import { prisma } from "@/lib/prisma";
import { ERROR_MESSAGES } from "@/lib/utils/errorMessages";
```

## 🔐 Authentication & Data Scoping

### Business Rules for Doctor-Patient Access
- **Doctors**: Can ONLY see their own `DoctorPatient` records
- **Critical**: Doctors CANNOT access registered `Patient` profiles
- **Data Scoping**: ALL queries must filter by doctor/user ID

```typescript
// ✅ CORRECT: Proper doctor data scoping
const { userId } = await auth();
const doctorProfile = await prisma.doctorProfile.findUnique({
  where: { userId }
});

const patients = await prisma.doctorPatient.findMany({
  where: { doctorId: doctorProfile.id } // ALWAYS scope to doctor
});

// ❌ WRONG: Unscoped data access - SECURITY RISK
const patients = await prisma.doctorPatient.findMany(); // Missing doctor filter
```

### Role-Based Access Patterns
```typescript
// ✅ CORRECT: Server component with proper access control
export async function PatientTableSC() {
  const { userId } = await auth();
  if (!userId) redirect("/sign-in");
  
  const doctorProfile = await prisma.doctorProfile.findUnique({
    where: { userId },
    select: { id: true }
  });
  
  if (!doctorProfile) redirect("/register/doctor");
  
  // Data scoped to this doctor only
  const patients = await prisma.doctorPatient.findMany({
    where: { doctorId: doctorProfile.id }
  });
  
  return <PatientTable patients={patients} />;
}
```

## 📁 File Organization Requirements

### Component Structure Standards
```
app/[role]/[feature]/
├── components/              # Feature-specific components
│   ├── FeatureSC.tsx       # Server component (data fetching)
│   ├── FeatureTable.tsx    # Client component (UI)
│   ├── FeatureModal.tsx    # Client component (interactions)
│   └── FeatureRow.tsx      # Client component (individual items)
├── [id]/                   # Dynamic routes
│   ├── page.tsx            # Route page
│   ├── loading.tsx         # Loading state (REQUIRED)
│   └── error.tsx           # Error state (REQUIRED)
├── page.tsx                # Main feature page
├── loading.tsx             # Feature loading state (REQUIRED)
└── error.tsx              # Feature error state (REQUIRED)
```

### Utility Organization
```
lib/utils/
├── errorMessages.ts        # ALL Spanish error messages
├── dateUtils.ts           # Date formatting utilities
├── validation.ts          # Shared validation logic
└── constants.ts           # Application constants
```

## 🔧 Code Quality Standards

### File Size Limits
- **Maximum**: 350 lines per component file
- **Action Required**: Extract utilities and split components when approaching limit
- **Exception**: Only with explicit justification and documentation

### Form Validation Pattern
```typescript
// ✅ REQUIRED: Client-side validation before API submission
const validateForm = () => {
  const errors: Record<string, string> = {};
  
  if (!form.fullName.trim()) {
    errors.fullName = ERROR_MESSAGES.FULL_NAME_REQUIRED;
  }
  
  if (form.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
    errors.email = ERROR_MESSAGES.INVALID_EMAIL;
  }
  
  setValidationErrors(errors);
  return Object.keys(errors).length === 0;
};

const handleSubmit = (formData) => {
  if (!validateForm()) return; // Validate BEFORE submission
  
  startTransition(async () => {
    const response = await fetch('/api/endpoint', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });
    
    if (response.ok) {
      router.refresh(); // Revalidate server component data
    } else {
      const result = await response.json();
      setError(result.error || ERROR_MESSAGES.UNEXPECTED_ERROR);
    }
  });
};
```

### API Route Standards
```typescript
import { NextRequest, NextResponse } from "next/server";
import { auth } from "@clerk/nextjs/server";
import { prisma } from "@/lib/prisma";
import { z } from "zod";
import { ERROR_MESSAGES } from "@/lib/utils/errorMessages";

// ✅ REQUIRED: Zod schema with centralized error messages
const Schema = z.object({
  fullName: z.string().min(1, ERROR_MESSAGES.FULL_NAME_REQUIRED),
  email: z.string().email(ERROR_MESSAGES.INVALID_EMAIL).optional().or(z.literal(""))
});

export async function POST(request: NextRequest) {
  try {
    // ✅ REQUIRED: Authentication check
    const { userId } = await auth();
    if (!userId) {
      return NextResponse.json(
        { error: ERROR_MESSAGES.UNAUTHORIZED }, 
        { status: 401 }
      );
    }

    // ✅ REQUIRED: Validation with Zod
    const body = await request.json();
    const validatedData = Schema.parse(body);
    
    // ✅ REQUIRED: User/role scoping
    const doctorProfile = await prisma.doctorProfile.findUnique({
      where: { userId },
      select: { id: true }
    });
    
    if (!doctorProfile) {
      return NextResponse.json(
        { error: ERROR_MESSAGES.DOCTOR_PROFILE_NOT_FOUND },
        { status: 404 }
      );
    }

    // ✅ REQUIRED: Scoped data operation
    const result = await prisma.doctorPatient.create({
      data: {
        ...validatedData,
        doctorId: doctorProfile.id // Always scope to authenticated user
      }
    });

    return NextResponse.json({ result }, { status: 201 });

  } catch (error) {
    // ✅ REQUIRED: Proper error handling with centralized messages
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { 
          error: ERROR_MESSAGES.VALIDATION_FAILED, 
          details: error.errors 
        },
        { status: 400 }
      );
    }
    
    console.error("API Error:", error);
    return NextResponse.json(
      { error: ERROR_MESSAGES.UNEXPECTED_ERROR },
      { status: 500 }
    );
  }
}
```

## 🎨 UI/UX Standards

### Material-UI Integration
- **Loading States**: Use MUI Skeleton components for better perceived performance
- **Spanish Text**: All labels, buttons, descriptions in Spanish
- **Form Patterns**: Client-side validation with immediate feedback

```typescript
// ✅ REQUIRED: Loading states with MUI Skeletons
export default function Loading() {
  return (
    <Container maxWidth="xl">
      <Skeleton variant="text" width="40%" height={40} />
      <Skeleton variant="rectangular" width="100%" height={400} />
    </Container>
  );
}

// ✅ REQUIRED: Spanish UI with validation feedback
<TextField
  label="Nombre Completo"
  placeholder="Ingresa el nombre del paciente"
  value={form.fullName}
  onChange={(e) => setForm(prev => ({ ...prev, fullName: e.target.value }))}
  error={!!validationErrors.fullName}
  helperText={validationErrors.fullName}
  required
/>
```

## 📚 Documentation Requirements

### For Major Changes (> 5 files or architectural changes)
Create comprehensive documentation in `.docs/YYYYMMDD/feature-name/`:
- `README.md` - Main overview with architecture diagrams
- `technical-changes.md` - Detailed before/after code analysis  
- `quick-reference.md` - Developer cheat sheet
- `index.md` - Navigation and summary

**Note**: Use descriptive feature names (e.g., `patient-management`, `appointment-scheduling`, `notes-system`) to avoid collisions when merging branches.

### Commit Message Standards
- **Searchable Keywords**: Include terms for AI agent discoverability
- **Structured Format**: Clear sections with technical context
- **Code Examples**: Before/after snippets showing changes
- **Migration Guidance**: Notes for similar future work

## ❌ Prohibited Patterns

### Never Use These Patterns
1. **Server Actions for Mutations** - Use API routes instead
2. **Custom Type Abstractions over Prisma** - Use Prisma types directly
3. **Hardcoded Spanish Strings** - Use ERROR_MESSAGES constants
4. **Fetch in Server Components** - Use direct Prisma calls
5. **Missing Loading/Error States** - Every route needs both
6. **Unscoped Data Access** - Always filter by user/role
7. **Files Over 350 Lines** - Extract components/utilities
8. **English UI Text** - All user-facing text in Spanish
9. **Mixed Language Code** - Code in English, UI in Spanish
10. **Any Types in TypeScript** - Use proper typing

### Code Review Red Flags
```typescript
// ❌ These patterns will be rejected in code review:

// Server actions
"use server";
async function createPatient() {} 

// Custom type wrappers
export type PatientData = DoctorPatient;

// Hardcoded Spanish
{ error: "Error al crear paciente" }

// Fetch in server component
const data = await fetch('/api/patients');

// Unscoped queries
const patients = await prisma.doctorPatient.findMany();

// English UI text
<Button>Create Patient</Button>

// Any types
const handleData = (data: any) => {}
```

## ✅ Success Criteria Checklist

Before submitting any code, verify:
- [ ] API routes used for all mutations (no server actions)
- [ ] Direct Prisma types used (no custom type abstractions)
- [ ] All UI text in Spanish using ERROR_MESSAGES constants
- [ ] Proper authentication and data scoping implemented
- [ ] Loading.tsx and error.tsx files created for all routes
- [ ] Client-side validation before API submission
- [ ] TypeScript compilation passes with no errors
- [ ] Files under 350 lines (or justified exception)
- [ ] Pre-commit hooks pass (linting, formatting, type checking)
- [ ] Major changes include comprehensive documentation

## 🎯 Eduardo's Philosophy

**Quality over Speed**: Eduardo prioritizes maintainable, well-documented code over quick implementations. When in doubt:
1. Choose the more maintainable solution
2. Document architectural decisions
3. Implement comprehensive error handling
4. Ensure type safety throughout
5. Create detailed commit messages for future reference

**Mobile-First API Design**: All architectural decisions should consider future mobile app requirements, which is why API routes are preferred over server actions.

**Developer Experience**: Code should be self-documenting with clear patterns, comprehensive types, and helpful error messages for both developers and users.