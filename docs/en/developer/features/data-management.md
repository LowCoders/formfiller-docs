# Data Management

## Overview

The FormFiller data management system is responsible for validating, storing, querying, and exporting data from forms. The central component is `DataService`, which integrates validation and database operations.

## Data Model

### Structure

```typescript
interface IData extends Document {
  configId: ObjectId;        // Related form configuration
  userId?: ObjectId;         // Submitting user (optional)
  siteId?: ObjectId;         // Multisite tenant (if any)
  data: Record<string, any>; // Submitted data
  computedResults?: Record<string, any>; // Computed results
  workflow?: any[];          // Workflow execution log
  isActive: boolean;         // Soft delete flag
  createdAt: Date;
  updatedAt: Date;
}
```

### Database Indexes

```javascript
// Optimization for fast queries
dataSchema.index({ configId: 1, createdAt: -1 });
dataSchema.index({ userId: 1, configId: 1 });
dataSchema.index({ siteId: 1, isActive: 1 });
dataSchema.index({ createdAt: -1 });
```

## Save Process

### Code Example

```typescript
// DataService.createData() simplified version
async createData(
  configId: string, 
  userId: string | undefined, 
  data: Record<string, any>, 
  locale: string = 'en'
): Promise<{ data: IData; computedResults?: Record<string, any> }> {
  
  // 1. Config loading
  const config = await Config.findById(configId)
    .select('preferences siteId')
    .lean();
    
  if (!config) {
    throw new AppError('Configuration not found', 404);
  }

  // 2. Validation
  const validationService = new ValidationService();
  const validationResult = await validationService.validateFormData(
    configId,
    data,
    { locale, mode: 'sequential' }
  );

  if (!validationResult.valid) {
    throw new AppError('Validation failed', 400, {
      validationErrors: validationResult.errors
    });
  }

  // 3. Save limit check
  if (config.preferences?.saveLimit && userId) {
    const count = await Data.countDocuments({
      configId, userId, isActive: true
    });
    if (count >= config.preferences.saveLimit) {
      throw new AppError('Save limit reached', 400);
    }
  }

  // 4. Save
  const dataEntry = new Data({
    configId,
    userId,
    data,
    isActive: true,
    ...(config.siteId && { siteId: config.siteId }),
    ...(validationResult.computedResults && { 
      computedResults: validationResult.computedResults 
    })
  });

  const savedData = await dataEntry.save();
  
  return {
    data: savedData,
    computedResults: validationResult.computedResults
  };
}
```

## Queries

### Basic Queries

```typescript
// All data for a form
const results = await dataService.findAll({
  configId: 'config-id',
  isActive: true
});

// User's own data
const myData = await dataService.findAll({
  userId: 'user-id',
  configId: 'config-id',
  isActive: true
});

// Get single data
const data = await dataService.findById('data-id');
```

## Export

### Supported Formats

| Format | MIME Type | Usage |
|--------|-----------|-------|
| JSON | `application/json` | API integration, import |
| CSV | `text/csv` | Excel, spreadsheet apps |
| Excel | `application/vnd.openxmlformats-...` | Business reports |

### Export API

```typescript
// GET /api/data/:configId/export?format=csv&filters=...

async exportData(
  configId: string,
  format: 'json' | 'csv' | 'excel',
  filters?: Record<string, any>
): Promise<Buffer | object[]> {
  
  const data = await this.findAll({
    configId,
    isActive: true,
    ...filters
  });
  
  switch (format) {
    case 'json':
      return data;
      
    case 'csv':
      return this.convertToCsv(data);
      
    case 'excel':
      return this.convertToExcel(data);
  }
}
```

## Save Limit

### Configuration

In form configuration, you can set how many times a user can submit the form:

```json
{
  "preferences": {
    "saveLimit": 1,           // Max 1 submission
    "allowEdit": true,        // Editing allowed
    "allowDelete": false      // Deletion disabled
  }
}
```

### Save Limit Types

| Value | Behavior |
|-------|----------|
| `null` or `undefined` | Unlimited submissions |
| `1` | Single submission (e.g., application, voting) |
| `N` | Maximum N submissions (e.g., weekly reports) |

## ComputedResults Storage

### Enabling

```json
{
  "preferences": {
    "storeComputedResults": true
  }
}
```

### Use Cases

| Case | Example |
|------|---------|
| **Exam scoring** | Total score, partial results storage |
| **Calculator** | Calculated values (price, quantity) |
| **Risk assessment** | Risk score, category |
| **Classification** | Automatic categorization result |

## Soft Delete

### Operation

Instead of physically deleting records, we set `isActive: false`:

```typescript
async deleteData(dataId: string, userId: string): Promise<IData | null> {
  const data = await Data.findById(dataId);
  
  if (!data) {
    throw new AppError('Data not found', 404);
  }
  
  // Permission check
  if (data.userId?.toString() !== userId) {
    throw new AppError('Not authorized to delete this data', 403);
  }
  
  // Soft delete
  data.isActive = false;
  return data.save();
}
```

### Advantages

- **Recoverability**: Accidental deletion can be undone
- **Audit trail**: All data is preserved
- **Referential integrity**: References don't break

## API Endpoints

### Data Routes

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/data/:configId` | List data |
| GET | `/api/data/:configId/:dataId` | Get single data |
| POST | `/api/data/:configId` | Create new data |
| PUT | `/api/data/:configId/:dataId` | Update data |
| DELETE | `/api/data/:configId/:dataId` | Delete data (soft) |
| GET | `/api/data/:configId/export` | Export data |
| GET | `/api/data/:configId/stats` | Statistics |

## Related Documentation

- [Backend Development](../backend.md) - Backend architecture
- [Validation](../validation.md) - Validation rules
- [Workflow Management](./workflow.md) - Workflow integration
- [Schema](../schema.md) - Form configuration

