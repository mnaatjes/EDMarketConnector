# Notes on EDMC Refactor

## Architectural Goals

1. Organize and centralize logic into `src/` directory
  - Implement as Python modules
  - Segregate legacy architecture into modules
  - Professional Bootstrapping and Configuration 
  - Dependency Injection
  - Decouple Business Logic from other areas

2. Integrate `tests/` into CI/CD Pipeline
  - Consolidate legacy tests so they don't negatively impact upgraded CI/CD pipeline
  - Implement clear pytest implementation with mirrored src/ directory structures
  - Determine what additional test tooling is needed for external API interactions

3. CI/CD Pipeline Update
  - Begun with centralizing requirements.txt files in pyproject.toml
  - Integrate version control with git tags and CHANGELOG.md
  - Make pyproject.toml SSoT for version
  - Design distinct Dev, Stage, and Prod regimes

4. Application Modularity
  - Segregate legacy architecture into modules
  - Define necessary domains models
  - Separate GUI from logic
  - Decouple installation environment (Windows, Linux) from logic
  - Utilize API architecture for decoupling and communication accross modules where appropriate
  - List external dependencies (inara, eddn, other APIs)
  - List functionalities to preserve from legacy and reorganize them.

## Architectural Directives

1. Observe and maintain "Single Responsibility Principle"
2. Practice "Inversion of Control"
3. Ensure Maintainability and Scalability
4. Decouple Business Logic 
5. Keep GUI/Presentation-Layer clean of processing logic
6. Domain Driven Design and Event Sourcing
