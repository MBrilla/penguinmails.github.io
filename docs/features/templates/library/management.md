---
title: "Template Management"
description: "Template management and organization"
last_modified_date: "2025-12-04"
level: "3"
keywords: "management, organization, versioning"
---

# Template Management

## Level 3: Technical Implementation

### Database Schema

```sql
-- Template library (public templates)
CREATE TABLE template_library (
  id UUID PRIMARY KEY,

  -- Template metadata
  name VARCHAR(255) NOT NULL,
  description TEXT,
  category VARCHAR(100),
  subcategory VARCHAR(100),

  -- Template content
  html_content TEXT,
  json_structure JSONB,
  thumbnail_url TEXT,
  preview_images JSONB,  -- desktop, mobile, tablet

  -- Classification
  industry_tags TEXT[],
  complexity_level VARCHAR(50),  -- simple, moderate, advanced

  -- Customization
  customizable_components JSONB,
  brand_variables JSONB,  -- Which colors/fonts can be customized

  -- Status
  is_active BOOLEAN DEFAULT TRUE,
  is_featured BOOLEAN DEFAULT FALSE,
  is_premium BOOLEAN DEFAULT FALSE,

  -- Analytics
  usage_count INTEGER DEFAULT 0,
  rating DECIMAL(3,2),
  rating_count INTEGER DEFAULT 0,

  -- Performance benchmarks
  avg_open_rate DECIMAL(5,2),
  avg_click_rate DECIMAL(5,2),
  avg_conversion_rate DECIMAL(5,2),

  -- Version
  version VARCHAR(50),
  changelog TEXT,

  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_library_category ON template_library(category);
CREATE INDEX idx_library_industry ON template_library USING GIN(industry_tags);
CREATE INDEX idx_library_active ON template_library(is_active);
CREATE INDEX idx_library_featured ON template_library(is_featured);

-- Template usage tracking
CREATE TABLE template_library_usage (
  id UUID PRIMARY KEY,
  library_template_id UUID NOT NULL REFERENCES template_library(id),
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),

  -- Created template
  template_id UUID REFERENCES templates(id),

  -- Customizations applied
  customizations JSONB,

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_library_usage_template ON template_library_usage(library_template_id);
CREATE INDEX idx_library_usage_tenant ON template_library_usage(tenant_id);

-- Template ratings
CREATE TABLE template_ratings (
  id UUID PRIMARY KEY,
  library_template_id UUID NOT NULL REFERENCES template_library(id),
  user_id UUID NOT NULL REFERENCES users(id),

  rating INTEGER CHECK (rating >= 1 AND rating <= 5),
  review TEXT,

  created_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(library_template_id, user_id)
);

CREATE INDEX idx_ratings_template ON template_ratings(library_template_id);

-- Brand presets
CREATE TABLE brand_presets (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  workspace_id UUID REFERENCES workspaces(id),

  name VARCHAR(255) NOT NULL,

  -- Brand settings
  colors JSONB,
  typography JSONB,
  spacing JSONB,
  logo_url TEXT,

  -- Social media
  social_links JSONB,

  is_default BOOLEAN DEFAULT FALSE,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_brand_presets_tenant ON brand_presets(tenant_id);

```

### Template Library Service

```typescript
interface TemplateLibraryFilter {
  category?: string;
  industryTags?: string[];
  complexityLevel?: 'simple' | 'moderate' | 'advanced';
  isPremium?: boolean;
  search?: string;
}

interface BrandPreset {
  colors: {
    primary: string;
    secondary: string;
    text: string;
    background: string;
  };
  typography: {
    headingFont: string;
    bodyFont: string;
  };
  spacing: {
    padding: number;
    margin: number;
  };
  logoUrl?: string;
  socialLinks?: Record<string, string>;
}

class TemplateLibraryService {
  async browseTemplates(
    filters: TemplateLibraryFilter,
    pagination: { offset: number; limit: number }
  ): Promise<{ templates: any[]; total: number }> {
    const query: any = {
      where: { isActive: true },
      offset: pagination.offset,
      limit: pagination.limit,
      order: [['usageCount', 'DESC']],
    };

    if (filters.category) {
      query.where.category = filters.category;
    }

    if (filters.industryTags && filters.industryTags.length > 0) {
      query.where.industryTags = {
        [Op.overlap]: filters.industryTags,
      };
    }

    if (filters.complexityLevel) {
      query.where.complexityLevel = filters.complexityLevel;
    }

    if (filters.isPremium !== undefined) {
      query.where.isPremium = filters.isPremium;
    }

    if (filters.search) {
      query.where[Op.or] = [
        { name: { [Op.iLike]: `%${filters.search}%` } },
        { description: { [Op.iLike]: `%${filters.search}%` } },
      ];
    }

    const templates = await db.templateLibrary.findAll(query);
    const total = await db.templateLibrary.count({ where: query.where });

    return { templates, total };
  }

  async useTemplate(
    libraryTemplateId: string,
    tenantId: string,
    workspaceId: string,
    options: {
      name: string;
      brandPreset?: BrandPreset;
    }
  ): Promise<Template> {
    const libraryTemplate = await db.templateLibrary.findById(libraryTemplateId);

    if (!libraryTemplate) {
      throw new Error('Template not found');
    }

    // Clone template structure
    let jsonStructure = JSON.parse(JSON.stringify(libraryTemplate.jsonStructure));

    // Apply brand preset if provided
    if (options.brandPreset) {
      jsonStructure = this.applyBrandPreset(jsonStructure, options.brandPreset);
    }

    // Create new template
    const template = await db.templates.create({
      tenantId,
      workspaceId,
      name: options.name,
      description: libraryTemplate.description,
      category: libraryTemplate.category,
      jsonStructure,
      htmlContent: libraryTemplate.htmlContent,
      status: 'draft',
      parentTemplateId: libraryTemplateId,
    });

    // Track usage
    await db.templateLibraryUsage.create({
      libraryTemplateId,
      tenantId,
      workspaceId,
      templateId: template.id,
      customizations: options.brandPreset,
    });

    // Increment usage count
    await db.templateLibrary.update(libraryTemplateId, {
      usageCount: libraryTemplate.usageCount + 1,
    });

    return template;
  }

  private applyBrandPreset(
    structure: any,
    preset: BrandPreset
  ): any {
    // Recursively apply brand colors and fonts
    const applyToComponent = (component: any) => {
      if (component.properties) {
        // Apply colors
        if (component.properties.backgroundColor === '{{brand.primary}}') {
          component.properties.backgroundColor = preset.colors.primary;
        }
        if (component.properties.textColor === '{{brand.text}}') {
          component.properties.textColor = preset.colors.text;
        }

        // Apply fonts
        if (component.properties.fontFamily === '{{brand.headingFont}}') {
          component.properties.fontFamily = preset.typography.headingFont;
        }

        // Apply logo
        if (component.type === 'image' && component.properties.src === '{{brand.logo}}') {
          component.properties.src = preset.logoUrl;
        }
      }

      // Recurse to children
      if (component.children) {
        component.children.forEach(applyToComponent);
      }
    };

    structure.components.forEach(applyToComponent);

    // Update global styles
    if (structure.globalStyles) {
      structure.globalStyles.colors = preset.colors;
      structure.globalStyles.fonts = {
        body: preset.typography.bodyFont,
        heading: preset.typography.headingFont,
      };
    }

    return structure;
  }

  async rateTemplate(
    libraryTemplateId: string,
    userId: string,
