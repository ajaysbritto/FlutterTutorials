# Technical Architecture: Project Context Memory System

## System Overview

The Project Context Memory System enables Cursor AI to maintain persistent, project-specific knowledge across development sessions. This system learns from corrections, stores domain expertise, and provides contextually relevant suggestions.

## Architecture Components

### 1. Memory Storage Layer

#### 1.1 Project Knowledge Database
```typescript
interface ProjectKnowledgeDB {
  projects: Map<string, ProjectKnowledge>;
  learnings: Map<string, Learning[]>;
  decisions: Map<string, Decision[]>;
  patterns: Map<string, Pattern[]>;
  antiPatterns: Map<string, AntiPattern[]>;
  businessRules: Map<string, BusinessRule[]>;
}

class ProjectKnowledgeManager {
  private db: ProjectKnowledgeDB;
  private vectorStore: VectorStore; // For semantic search
  private embeddingModel: EmbeddingModel;
  
  async storeLearning(projectId: string, learning: Learning): Promise<void> {
    // Store in database
    await this.db.learnings.get(projectId).push(learning);
    
    // Create embedding for semantic search
    const embedding = await this.embeddingModel.embed(learning.context + learning.solution);
    await this.vectorStore.store(projectId, learning.id, embedding);
    
    // Update project knowledge
    await this.updateProjectKnowledge(projectId);
  }
  
  async findRelevantLearnings(
    projectId: string, 
    query: string, 
    limit: number = 10
  ): Promise<Learning[]> {
    // Create query embedding
    const queryEmbedding = await this.embeddingModel.embed(query);
    
    // Search vector store
    const similarIds = await this.vectorStore.search(projectId, queryEmbedding, limit);
    
    // Retrieve full learning objects
    const learnings = await this.db.learnings.get(projectId);
    return similarIds.map(id => learnings.find(l => l.id === id)).filter(Boolean);
  }
}
```

#### 1.2 Vector Store for Semantic Search
```typescript
interface VectorStore {
  store(projectId: string, id: string, embedding: number[]): Promise<void>;
  search(projectId: string, queryEmbedding: number[], limit: number): Promise<string[]>;
  update(projectId: string, id: string, embedding: number[]): Promise<void>;
  delete(projectId: string, id: string): Promise<void>;
}

class ChromaVectorStore implements VectorStore {
  private client: ChromaClient;
  
  async store(projectId: string, id: string, embedding: number[]): Promise<void> {
    await this.client.add({
      collection: `project_${projectId}`,
      documents: [id],
      embeddings: [embedding],
      metadatas: [{ projectId, id }]
    });
  }
  
  async search(projectId: string, queryEmbedding: number[], limit: number): Promise<string[]> {
    const results = await this.client.query({
      collection: `project_${projectId}`,
      queryEmbeddings: [queryEmbedding],
      nResults: limit
    });
    
    return results.ids[0] as string[];
  }
}
```

### 2. Learning System

#### 2.1 Correction Learning Engine
```typescript
class CorrectionLearningEngine {
  private memoryManager: ProjectMemoryManager;
  private patternRecognizer: PatternRecognizer;
  
  async learnFromCorrection(
    projectId: string,
    originalSuggestion: string,
    correction: string,
    context: string,
    reasoning?: string
  ): Promise<void> {
    // Create learning from correction
    const learning: Learning = {
      id: generateId(),
      type: this.inferLearningType(context),
      context: context,
      solution: correction,
      reasoning: reasoning || `Corrected from: ${originalSuggestion}`,
      confidence: 0.8,
      frequency: 1,
      lastUsed: new Date(),
      source: 'correction'
    };
    
    // Store learning
    await this.memoryManager.storeLearning(projectId, learning);
    
    // Create anti-pattern from original suggestion
    const antiPattern: AntiPattern = {
      id: generateId(),
      name: `Rejected: ${this.extractPatternName(originalSuggestion)}`,
      description: originalSuggestion,
      whyRejected: `User corrected to: ${correction}`,
      alternatives: [correction],
      frequency: 1,
      lastRejected: new Date()
    };
    
    await this.memoryManager.storeAntiPattern(projectId, antiPattern);
    
    // Check for pattern consolidation
    await this.patternRecognizer.consolidatePatterns(projectId, learning);
  }
  
  private inferLearningType(context: string): LearningType {
    if (context.includes('API') || context.includes('library')) return 'technical';
    if (context.includes('architecture') || context.includes('design')) return 'architectural';
    if (context.includes('business') || context.includes('domain')) return 'domain';
    return 'technical';
  }
}
```

#### 2.2 Pattern Recognition System
```typescript
class PatternRecognizer {
  private memoryManager: ProjectMemoryManager;
  private similarityThreshold = 0.7;
  
  async consolidatePatterns(projectId: string, newLearning: Learning): Promise<void> {
    const existingLearnings = await this.memoryManager.getLearnings(projectId);
    
    // Find similar learnings
    const similarLearnings = existingLearnings.filter(learning => 
      this.calculateSimilarity(learning, newLearning) > this.similarityThreshold
    );
    
    if (similarLearnings.length > 0) {
      // Consolidate with most similar learning
      const mostSimilar = similarLearnings.reduce((best, current) => 
        this.calculateSimilarity(current, newLearning) > 
        this.calculateSimilarity(best, newLearning) ? current : best
      );
      
      await this.consolidateLearnings(projectId, mostSimilar, newLearning);
    }
  }
  
  private calculateSimilarity(learning1: Learning, learning2: Learning): number {
    // Use embedding similarity for semantic comparison
    const embedding1 = this.getEmbedding(learning1);
    const embedding2 = this.getEmbedding(learning2);
    
    return this.cosineSimilarity(embedding1, embedding2);
  }
  
  private async consolidateLearnings(
    projectId: string, 
    existing: Learning, 
    newLearning: Learning
  ): Promise<void> {
    // Update existing learning with new information
    existing.frequency += 1;
    existing.confidence = Math.min(existing.confidence + 0.1, 1.0);
    existing.lastUsed = new Date();
    
    // Merge reasoning
    existing.reasoning += `; Also: ${newLearning.reasoning}`;
    
    // Update in database
    await this.memoryManager.updateLearning(projectId, existing);
    
    // Remove new learning (it's been consolidated)
    await this.memoryManager.removeLearning(projectId, newLearning.id);
  }
}
```

### 3. Context-Aware Suggestion Engine

#### 3.1 Contextual Suggestion Generator
```typescript
class ContextualSuggestionEngine {
  private memoryManager: ProjectMemoryManager;
  private antiPatternDetector: AntiPatternDetector;
  private businessRuleEngine: BusinessRuleEngine;
  
  async generateSuggestion(
    projectId: string,
    problem: string,
    context: string,
    codeContext?: string
  ): Promise<Suggestion> {
    // Get relevant learnings
    const learnings = await this.memoryManager.findRelevantLearnings(
      projectId, 
      problem + ' ' + context, 
      5
    );
    
    // Get anti-patterns to avoid
    const antiPatterns = await this.memoryManager.findRelevantAntiPatterns(
      projectId, 
      problem + ' ' + context, 
      3
    );
    
    // Get applicable business rules
    const businessRules = await this.businessRuleEngine.getApplicableRules(
      projectId, 
      context
    );
    
    // Generate suggestion
    const suggestion = await this.generateFromContext(
      problem,
      learnings,
      antiPatterns,
      businessRules,
      codeContext
    );
    
    return suggestion;
  }
  
  private async generateFromContext(
    problem: string,
    learnings: Learning[],
    antiPatterns: AntiPattern[],
    businessRules: BusinessRule[],
    codeContext?: string
  ): Promise<Suggestion> {
    // Use LLM to generate suggestion based on context
    const prompt = this.buildContextualPrompt(
      problem,
      learnings,
      antiPatterns,
      businessRules,
      codeContext
    );
    
    const response = await this.llm.generate(prompt);
    
    return {
      solution: response.solution,
      reasoning: response.reasoning,
      confidence: response.confidence,
      basedOn: learnings.map(l => l.id),
      avoids: antiPatterns.map(a => a.id),
      follows: businessRules.map(r => r.id)
    };
  }
}
```

#### 3.2 Anti-Pattern Detection
```typescript
class AntiPatternDetector {
  private memoryManager: ProjectMemoryManager;
  
  async detectAntiPatterns(
    projectId: string,
    suggestion: string,
    context: string
  ): Promise<AntiPatternMatch[]> {
    const antiPatterns = await this.memoryManager.getAntiPatterns(projectId);
    const matches: AntiPatternMatch[] = [];
    
    for (const antiPattern of antiPatterns) {
      const similarity = await this.calculateSimilarity(suggestion, antiPattern.description);
      
      if (similarity > 0.8) {
        matches.push({
          antiPattern,
          similarity,
          reason: antiPattern.whyRejected,
          alternatives: antiPattern.alternatives
        });
      }
    }
    
    return matches;
  }
}
```

### 4. Domain-Specific Knowledge Management

#### 4.1 Business Rule Engine
```typescript
class BusinessRuleEngine {
  private memoryManager: ProjectMemoryManager;
  
  async getApplicableRules(projectId: string, context: string): Promise<BusinessRule[]> {
    const allRules = await this.memoryManager.getBusinessRules(projectId);
    
    return allRules.filter(rule => 
      this.isRuleApplicable(rule, context)
    );
  }
  
  private isRuleApplicable(rule: BusinessRule, context: string): boolean {
    // Check if rule keywords match context
    const ruleKeywords = rule.keywords || [];
    const contextLower = context.toLowerCase();
    
    return ruleKeywords.some(keyword => 
      contextLower.includes(keyword.toLowerCase())
    );
  }
  
  async validateSolution(
    projectId: string,
    solution: string,
    context: string
  ): Promise<ValidationResult> {
    const applicableRules = await this.getApplicableRules(projectId, context);
    const violations: RuleViolation[] = [];
    
    for (const rule of applicableRules) {
      if (!this.solutionFollowsRule(solution, rule)) {
        violations.push({
          rule,
          violation: `Solution violates: ${rule.description}`,
          suggestion: rule.suggestion
        });
      }
    }
    
    return {
      valid: violations.length === 0,
      violations,
      score: 1 - (violations.length / applicableRules.length)
    };
  }
}
```

#### 4.2 Domain Knowledge Extractor
```typescript
class DomainKnowledgeExtractor {
  async extractDomainKnowledge(
    projectId: string,
    conversation: Conversation[]
  ): Promise<DomainKnowledge[]> {
    const domainKnowledge: DomainKnowledge[] = [];
    
    for (const message of conversation) {
      if (this.isDomainKnowledge(message.content)) {
        const knowledge = await this.parseDomainKnowledge(message.content);
        domainKnowledge.push(knowledge);
      }
    }
    
    return domainKnowledge;
  }
  
  private isDomainKnowledge(content: string): boolean {
    // Look for domain-specific patterns
    const domainPatterns = [
      /we learned that/i,
      /we established that/i,
      /the rule is/i,
      /in this domain/i,
      /for this project/i
    ];
    
    return domainPatterns.some(pattern => pattern.test(content));
  }
  
  private async parseDomainKnowledge(content: string): Promise<DomainKnowledge> {
    // Use NLP to extract structured knowledge
    const entities = await this.extractEntities(content);
    const relationships = await this.extractRelationships(content);
    
    return {
      id: generateId(),
      content,
      entities,
      relationships,
      confidence: 0.8,
      extractedAt: new Date()
    };
  }
}
```

### 5. Memory Management and Optimization

#### 5.1 Memory Consolidation
```typescript
class MemoryConsolidator {
  private memoryManager: ProjectMemoryManager;
  
  async consolidateProjectMemory(projectId: string): Promise<void> {
    const project = await this.memoryManager.getProject(projectId);
    
    // Consolidate similar learnings
    await this.consolidateLearnings(projectId);
    
    // Remove outdated information
    await this.removeOutdatedKnowledge(projectId);
    
    // Optimize memory usage
    await this.optimizeMemoryUsage(projectId);
  }
  
  private async consolidateLearnings(projectId: string): Promise<void> {
    const learnings = await this.memoryManager.getLearnings(projectId);
    const clusters = await this.clusterSimilarLearnings(learnings);
    
    for (const cluster of clusters) {
      if (cluster.length > 1) {
        await this.mergeLearningCluster(projectId, cluster);
      }
    }
  }
  
  private async removeOutdatedKnowledge(projectId: string): Promise<void> {
    const cutoffDate = new Date(Date.now() - 90 * 24 * 60 * 60 * 1000); // 90 days
    
    const learnings = await this.memoryManager.getLearnings(projectId);
    const outdated = learnings.filter(l => 
      l.lastUsed < cutoffDate && l.frequency < 2
    );
    
    for (const learning of outdated) {
      await this.memoryManager.removeLearning(projectId, learning.id);
    }
  }
}
```

#### 5.2 Context Relevance Scoring
```typescript
class ContextRelevanceScorer {
  async scoreRelevance(
    projectId: string,
    learning: Learning,
    currentContext: string
  ): Promise<number> {
    let score = 0;
    
    // Recency score (0-0.3)
    const daysSinceLastUsed = (Date.now() - learning.lastUsed.getTime()) / (1000 * 60 * 60 * 24);
    const recencyScore = Math.max(0, 0.3 - (daysSinceLastUsed / 30));
    score += recencyScore;
    
    // Frequency score (0-0.3)
    const frequencyScore = Math.min(0.3, learning.frequency / 10);
    score += frequencyScore;
    
    // Semantic similarity score (0-0.4)
    const similarity = await this.calculateSemanticSimilarity(
      learning.context + learning.solution,
      currentContext
    );
    score += similarity * 0.4;
    
    return Math.min(1, score);
  }
}
```

## Data Flow

### 1. Learning from Corrections
```
User Correction → CorrectionLearningEngine → Store Learning + AntiPattern → PatternRecognizer → Consolidate if Similar
```

### 2. Generating Suggestions
```
User Query → ContextualSuggestionEngine → Find Relevant Learnings → Check AntiPatterns → Apply Business Rules → Generate Suggestion
```

### 3. Memory Consolidation
```
Scheduled Task → MemoryConsolidator → Cluster Similar Learnings → Remove Outdated → Optimize Storage
```

## Performance Considerations

### 1. Caching Strategy
- Cache frequently accessed learnings
- Use Redis for hot data
- Implement LRU eviction policy

### 2. Search Optimization
- Use vector similarity search
- Implement query caching
- Batch operations when possible

### 3. Storage Optimization
- Compress stored data
- Use efficient serialization
- Implement data archiving

## Security and Privacy

### 1. Data Isolation
- Project-specific data isolation
- User access controls
- Encryption at rest

### 2. Sensitive Data Filtering
- Filter out sensitive information
- User control over what's stored
- Data retention policies

### 3. Compliance
- GDPR compliance
- Data deletion capabilities
- Audit logging

This architecture provides a robust foundation for maintaining project-specific context and learning from user interactions, addressing the core issue of Cursor AI "forgetting" established patterns and domain knowledge.