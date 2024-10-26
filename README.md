# standard-routes
A standard for payment codes of financial institutions

## Usage
```typescript
// Types for our Standard Routes system
type Institution = {
  id: string;
  name: string;
  short_name: string;
  type: string;
  legacy: {
    bank_code: string;
  };
  status: string;
};

type StandardRoutesData = {
  country: {
    name: string;
    iso: string;
    currency: string;
  };
  institutions: {
    list: Institution[];
  };
};

class StandardRoutesManager {
  private institutions: Institution[];
  
  constructor(data: StandardRoutesData) {
    this.institutions = data.institutions.list;
  }

  // Find institution by Standard Routes ID
  findByStandardCode(standardCode: string): Institution | null {
    return this.institutions.find(inst => inst.id === standardCode) || null;
  }

  // Find institution by legacy code
  findByLegacyCode(legacyCode: string): Institution | null {
    return this.institutions.find(inst => inst.legacy.bank_code === legacyCode) || null;
  }

  // Convert Standard Routes ID to legacy code
  toLegacyCode(standardCode: string): string | null {
    const institution = this.findByStandardCode(standardCode);
    return institution?.legacy.bank_code || null;
  }

  // Convert legacy code to Standard Routes ID
  toStandardCode(legacyCode: string): string | null {
    const institution = this.findByLegacyCode(legacyCode);
    return institution?.id || null;
  }
}

// Usage Examples

// 1. Initialize with our data
import institutionsData from './nigeria.json';
const standardRoutes = new StandardRoutesManager(institutionsData);

// 2. Example: Processing a payment using Standard Routes ID
async function processPayment({
  standardRoutesId,  // e.g., "23401000005" (GTBank)
  accountNumber,
  amount,
  currency,
  reference,
  narration
}: {
  standardRoutesId: string;
  accountNumber: string;
  amount: number;
  currency: string;
  reference: string;
  narration: string;
}) {
  // Convert Standard Routes ID to legacy code
  const legacyCode = standardRoutes.toLegacyCode(standardRoutesId);
  
  if (!legacyCode) {
    throw new Error('Invalid institution code');
  }

  // Prepare payload for payment provider
  const paymentPayload = {
    account_bank: legacyCode,
    account_number: accountNumber,
    amount,
    currency,
    reference,
    narration
  };

  // Send to payment provider
  return await submitPayment(paymentPayload);
}

// 3. Example Usage with your specific case
const payment = {
  standardRoutesId: "23401000009", // GTBank's Standard Routes ID
  accountNumber: "1234858909",
  amount: 3500,
  currency: "NGN",
  reference: "TRF_129230",
  narration: "Weekend blues"
};

// 4. Real-world example with validation
async function validateAndProcessPayment(paymentData: typeof payment) {
  // Find institution
  const institution = standardRoutes.findByStandardCode(paymentData.standardRoutesId);
  
  if (!institution) {
    throw new Error('Invalid institution');
  }

  if (institution.status !== 'active') {
    throw new Error('Institution not active');
  }

  // Log for debugging
  console.log(`Converting ${institution.name} (${institution.id}) to legacy code ${institution.legacy.bank_code}`);

  // Process payment
  return await processPayment(paymentData);
}

// 5. Example with error handling
try {
  await validateAndProcessPayment({
    standardRoutesId: "23401000009", // GTBank
    accountNumber: "1234858909",
    amount: 3500,
    currency: "NGN",
    reference: "TRF_129230",
    narration: "Weekend blues"
  });
} catch (error) {
  console.error('Payment failed:', error.message);
}

// 6. Utility function for bank selection UI
function getInstitutionsByType(type: string) {
  return institutionsData.institutions.list
    .filter(inst => inst.type === type)
    .map(inst => ({
      id: inst.id,
      name: inst.name,
      shortName: inst.short_name
    }));
}

// Example: Get all commercial banks
const commercialBanks = getInstitutionsByType('commercial_bank');

// 7. Validation utility
function validateBankCode(standardCode: string) {
  const institution = standardRoutes.findByStandardCode(standardCode);
  return {
    isValid: !!institution,
    institution,
    legacyCode: institution?.legacy.bank_code
  };
}
```
