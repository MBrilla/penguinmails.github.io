---
title: Feature Categories
description: Implementation patterns for UI/UX, API, and infrastructure features
last_modified_date: 2025-01-15
level: 3
persona: ["developer"]
keywords: [ui-ux, api, infrastructure, patterns, react, typescript]
---

# Feature Categories

Implementation patterns and examples for different types of features in PenguinMails.

## UI/UX Improvements

React component patterns for user interface features.

```typescript
// components/CampaignOptimization.tsx
interface OptimizationPanelProps {
  readonly campaign: EmailCampaign;
  readonly onOptimizationApplied: (optimizedContent: EmailContent) => void;
}

export const CampaignOptimizationPanel: React.FC<OptimizationPanelProps> = ({
  campaign,
  onOptimizationApplied
}) => {
  const [isOptimizing, setIsOptimizing] = useState(false);
  const [optimizationScore, setOptimizationScore] = useState<number | null>(null);
  const [recommendations, setRecommendations] = useState<string[]>([]);

  const handleOptimize = useCallback(async () => {
    setIsOptimizing(true);
    try {
      const result = await aiService.optimizeCampaign(campaign.id);

      setOptimizationScore(result.score);
      setRecommendations(result.recommendations);

      if (result.confidence > 0.8) {
        onOptimizationApplied(result.optimizedContent);
      }
    } catch (error) {
      toast.error('Optimization failed. Please try again.');
    } finally {
      setIsOptimizing(false);
    }
  }, [campaign.id, aiService, onOptimizationApplied]);

  return (
    <div className="optimization-panel">
      <div className="panel-header">
        <h3>AI Content Optimization</h3>
        <Button
          onClick={handleOptimize}
          disabled={isOptimizing}
          loading={isOptimizing}
        >
          {isOptimizing ? 'Optimizing...' : 'Optimize with AI'}
        </Button>
      </div>

      {optimizationScore !== null && (
        <div className="optimization-results">
          <div className="score-display">
            <span className="score-label">Optimization Score:</span>
            <span className="score-value">{optimizationScore}%</span>
          </div>

          <div className="recommendations">
            <h4>Recommendations</h4>
            <ul>
              {recommendations.map((rec, index) => (
                <li key={index}>{rec}</li>
              ))}
            </ul>
          </div>
        </div>
      )}
    </div>
  );
};
```

## API Enhancements

Endpoint design patterns for API features.

```typescript
// api/ai-optimization.ts
export class AIOptimizationAPI {
  constructor(private readonly httpClient: HTTPClient) {}

  async optimizeContent(request: OptimizationRequest): Promise<OptimizationResponse> {
    return this.httpClient.post('/api/v1/ai/optimize-content', {
      ...request,
      options: {
        confidence_threshold: 0.8,
        include_explanations: true,
        target_metrics: ['open_rate', 'click_rate']
      }
    });
  }

  async getOptimizationHistory(
    campaignId: string,
    options?: HistoryOptions
  ): Promise<OptimizationHistory[]> {
    return this.httpClient.get(`/api/v1/ai/optimization-history/${campaignId}`, {
      params: options
    });
  }

  async trainModel(datasets: TrainingDataset[]): Promise<TrainingResult> {
    return this.httpClient.post('/api/v1/ai/train-model', {
      datasets,
      hyperparameters: {
        learning_rate: 0.01,
        max_depth: 6,
        n_estimators: 100
      }
    });
  }
}

// Request/Response types
interface OptimizationRequest {
  readonly content: {
    readonly subject: string;
    readonly html: string;
    readonly text?: string;
  };
  readonly audience: {
    readonly demographics: AudienceDemographics;
    readonly preferences?: AudiencePreferences;
  };
  readonly constraints?: ContentConstraints;
}

interface OptimizationResponse {
  readonly optimized_content: {
    readonly subject: string;
    readonly html: string;
    readonly text?: string;
  };
  readonly improvement_score: number;
  readonly confidence: number;
  readonly recommendations: string[];
  readonly explanations: OptimizationExplanation[];
}
```

## Infrastructure Features

Database migration patterns for infrastructure features.

```typescript
// migrations/0024_add_ai_optimization_features.ts
import { sql } from 'postgres';

interface MigrationResult {
  success: boolean;
  message: string;
  tablesCreated?: string[];
  columnsAdded?: string[];
}

class AIOptimizationMigration implements DatabaseMigration {
  async upgrade(): Promise<MigrationResult> {
    try {
      const results = {
        success: true,
        message: 'AI optimization features migration completed',
        tablesCreated: [] as string[],
        columnsAdded: [] as string[]
      };

      // Create AI optimization cache table
      await sql`
        CREATE TABLE ai_optimization_cache (
          id SERIAL PRIMARY KEY,
          content_hash VARCHAR(64) UNIQUE NOT NULL,
          original_subject TEXT NOT NULL,
          optimized_subject TEXT NOT NULL,
          improvement_score FLOAT NOT NULL,
          confidence FLOAT NOT NULL,
          optimization_type VARCHAR(50) NOT NULL,
          created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
          expires_at TIMESTAMP NOT NULL
        );
      `;
      results.tablesCreated.push('ai_optimization_cache');

      // Create indexes
      await sql`CREATE INDEX ix_optimization_cache_content_hash 
                ON ai_optimization_cache(content_hash);`;
      await sql`CREATE INDEX ix_optimization_cache_expires 
                ON ai_optimization_cache(expires_at);`;

      // Create optimization metrics table
      await sql`
        CREATE TABLE optimization_metrics (
          id SERIAL PRIMARY KEY,
          campaign_id INTEGER NOT NULL,
          original_metrics JSONB NOT NULL,
          optimized_metrics JSONB NOT NULL,
          improvement_percentage FLOAT NOT NULL,
          metric_type VARCHAR(50) NOT NULL,
          created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
          FOREIGN KEY (campaign_id) REFERENCES campaigns(id) ON DELETE CASCADE
        );
      `;
      results.tablesCreated.push('optimization_metrics');

      // Add optimization columns to campaigns
      await sql`ALTER TABLE campaigns 
                ADD COLUMN ai_optimization_enabled BOOLEAN DEFAULT FALSE;`;
      await sql`ALTER TABLE campaigns 
                ADD COLUMN optimization_score FLOAT;`;
      await sql`ALTER TABLE campaigns 
                ADD COLUMN last_optimization TIMESTAMP;`;
      
      results.columnsAdded.push(
        'campaigns.ai_optimization_enabled',
        'campaigns.optimization_score',
        'campaigns.last_optimization'
      );

      return results;
    } catch (error) {
      return {
        success: false,
        message: `Migration failed: ${error instanceof Error ? error.message : 'Unknown error'}`
      };
    }
  }

  async downgrade(): Promise<MigrationResult> {
    try {
      // Remove columns from campaigns
      await sql`ALTER TABLE campaigns DROP COLUMN IF EXISTS last_optimization;`;
      await sql`ALTER TABLE campaigns DROP COLUMN IF EXISTS optimization_score;`;
      await sql`ALTER TABLE campaigns DROP COLUMN IF EXISTS ai_optimization_enabled;`;

      // Drop tables
      await sql`DROP TABLE IF EXISTS optimization_metrics CASCADE;`;
      await sql`DROP TABLE IF EXISTS ai_optimization_cache CASCADE;`;

      return {
        success: true,
        message: 'AI optimization features rollback completed'
      };
    } catch (error) {
      return {
        success: false,
        message: `Rollback failed: ${error instanceof Error ? error.message : 'Unknown error'}`
      };
    }
  }
}
```

## Migration Runner Utility

```typescript
class MigrationRunner {
  async runMigration(migration: DatabaseMigration): Promise<boolean> {
    try {
      console.log('Starting migration...');
      const result = await migration.upgrade();

      if (result.success) {
        console.log(`Migration completed: ${result.message}`);
        if (result.tablesCreated) {
          console.log(`Tables created: ${result.tablesCreated.join(', ')}`);
        }
        if (result.columnsAdded) {
          console.log(`Columns added: ${result.columnsAdded.join(', ')}`);
        }
        return true;
      } else {
        console.error(`Migration failed: ${result.message}`);
        return false;
      }
    } catch (error) {
      console.error(`Migration error: ${error instanceof Error ? error.message : 'Unknown error'}`);
      return false;
    }
  }

  async rollbackMigration(migration: DatabaseMigration): Promise<boolean> {
    try {
      console.log('Starting rollback...');
      const result = await migration.downgrade();

      if (result.success) {
        console.log(`Rollback completed: ${result.message}`);
        return true;
      }
      console.error(`Rollback failed: ${result.message}`);
      return false;
    } catch (error) {
      console.error(`Rollback error: ${error instanceof Error ? error.message : 'Unknown error'}`);
      return false;
    }
  }
}
```

## Related Documentation

- [Process & Templates](./process) - Feature request workflow
- [Implementation Standards](./implementation) - Coding patterns
- [Development Guidelines](./guidelines) - Quality gates and checklists
