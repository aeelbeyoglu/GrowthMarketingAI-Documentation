# Project Requirements Document (PRD)

## 1. Project Overview

The Growth Marketing API Project is built on Cloudflare Workers using TypeScript. It serves as a comprehensive API for marketing and SEO functionalities, including:

- Retrieval of Search Engine Results Page (SERP) data from Google, Bing, and YouTube (via DataForSEO).  
- Company information lookups and logo retrieval (integrations with Clearbit).  
- HTML Source Code Searches, enabling users to look for specific terms within the HTML of various domains.  
- Extensive logging, rate limiting, authentication, and error handling.  

Although part of your broader ecosystem may include Next.js 14 for front-end or other services, the codebase shown here implements edge-computing APIs in a Cloudflare Worker environment.

### Highlights

1. Runs on Cloudflare Workers (edge-computing model).  
2. Written entirely in TypeScript (ensures type safety across handlers, services, and utilities).  
3. Organized into distinct modules (e.g., handlers, services, middleware) for clarity and maintainability.  
4. Integrates with third-party providers like DataForSEO, Clearbit, and optionally KeywordsEverywhere.  
5. Provides robust logging, request validation, and rate-limiting capabilities.  
6. Designed for SEO and marketing purposes, focusing on SERP data retrieval, domain intelligence, HTML source code analysis, related keywords exploration, and analytics.  

This project allows you to integrate marketing data, such as SERP listings, keyword autocomplete suggestions, domain HTML search results, and video statistics, into your own applications with minimal overhead.

---

## 2. Core Functionalities

1. **SERP APIs (DataForSEO Integration):**  
   - Google SERP (Organic, Autocomplete)  
   - Bing SERP (Organic)  
   - YouTube SERP (Organic)  
   - YouTube Video Info  
   Each SERP flow is handled by methods in the DataForSEOService class (see src/services/dataforseo.ts). The service methods:
     - Accept typed parameters (e.g., GoogleAutocompleteRequest, BingSearchParams, YouTubeSerpRequest).  
     - Interact with the DataForSEO API endpoints (live/advanced) for real-time search data.  
     - Parse, validate, and return typed responses.  

2. **Company Logo & Information (Clearbit Integration):**  
   - A ClearbitService (see src/services/clearbit.ts) provides methods to:
     - Search for companies by name.  
     - Retrieve company logos by domain.  
     - Transform returned data (e.g., changing Clearbit’s logo URL to your own domain-based endpoint).  

3. **HTML Source Code Search (DataForSEO Integration):**  
   - Implemented by the “html-source-code-search.ts” handler under src/handlers/.  
   - Allows searching for user-supplied terms within the HTML of specified domains.  
   - Relies on DataForSEO’s HTML search capabilities.  
   - Logs search details and stores them in the request logs database.  
   - Usage: Clients send an array of search terms in the request body, optionally specifying result limits, offsets, and sort order.  

4. **Enriched Domain WHOIS (DataForSEO Integration):**  
   - Implemented by the “enriched-domain-whois.ts” handler under src/handlers/.  
   - Fetches domain WHOIS data plus additional SEO metrics (e.g., backlinks, domain rank).  
   - Useful for domain intelligence, ownership tracking, and competitor research.  
   - Integrates the data into typed structures for reliability.  

5. **Related Keywords (DataForSEO Integration):**  
   - Implemented by the “related-keywords.ts” handler under src/handlers/.  
   - Retrieves semantically related queries for a given seed keyword using DataForSEO’s depth-first search approach.  
   - Provides deeper keyword research capabilities, offering extended lists of synonyms, related terms, and search suggestions.  
   - Returns typed, structured responses to ensure consistency with other SERP-based responses.

6. **Rate Limiting & API Key Management:**  
   - The system enforces monthly quotas per API key.  
   - Provides usage endpoints (e.g., /api/keys/usage) and API key creation endpoints (e.g., POST /api/keys).  
   - Keeps track of rate limiting in D1Database.  

7. **Logging & Monitoring:**  
   - LoggingService (src/services/logging.ts) records each request’s endpoint, method, status, response time, and error messages.  
   - Logs are stored in a D1Database table called request_logs.  
   - Separate methods handle logging for different API functionalities (e.g., logCompanySearch, logMozMetrics, logSerpRequest, logDomainsByTechStack, etc.).  

8. **Middleware-Driven Router:**  
   - The code references router-based deployments via Hono or a custom router in src/core.  
   - Each route can apply different middleware layers (authentication, rate limiting, error handling, logging).  

9. **Error Handling & Validation:**  
   - Custom ApiError class used across services.  
   - Checks response status codes from external APIs.  
   - Provides a standardized JSON error response body.  

10. **Documentation & Type Definitions:**  
   - README.md provides high-level usage guides, endpoint definitions, and example requests/responses.  
   - .notes/project_overview.md adds context on current features and future roadmap.  
   - Type definitions in src/types ensure type-secure data flow for all major functionalities.  

---

## 3. Doc

Technical documentation is embedded via:

1. **.notes/project_overview.md**  
   - Summarizes the project’s vision, core features, and technical stack.  
   - Details the architecture (route handlers, service classes, utility functions).  

2. **README.md**  
   - Walks through the setup, structure, endpoints, authentication/rate limiting, and usage examples.  
   - Describes advanced features like historical data, error handling, and best practices.  

3. **Commented Methods and Interfaces**  
   - src/services/dataforseo.ts, src/services/clearbit.ts, and other files contain JSDoc-style comments describing the purpose, parameters, and return types of each function.  
   - src/types/* includes interfaces that capture the shape of external API responses (e.g., DataForSEOBingResponse, GoogleAutocompleteResponse, etc.), ensuring consistent and safe usage across the codebase.

4. **Error Responses**  
   - The stable, standardized error response shape (success: false, error, details) is documented in README.md.  

5. **Logging Statements**  
   - The LoggingService illustrates what details are captured (endpoint, method, IP, user-agent, etc.).  

6. **Future Roadmap**  
   - .notes/project_overview.md mentions planned features (additional search engine integrations, enhanced rate limiting, caching improvements).  

---

## 4. Current File Structure

Below is a representative breakdown reflecting both the README.md outline and the code snippets. The main application directory (“src/”) is organized into core, handlers, middleware, services, and types:

```
my_project/
├── .notes/
│   ├─ project_overview.md         # High-level overview and roadmap
│   └─ task_list.md                # Task management notes
├── src/
│   ├─ core/
│   │   ├─ router.ts               # Main router with middleware chaining
│   │   ├─ routes.ts               # Register routes and attach handlers
│   │   └─ middleware-manager.ts   # Middleware orchestration
│   ├─ config/
│   │   ├─ cors.ts                 # CORS configuration
│   │   └─ rate-limits.ts          # Rate limit settings
│   ├─ handlers/
│   │   ├─ company/
│   │   │   ├─ logo.ts             # Handle /logo/:domain requests
│   │   │   └─ search.ts           # Handle GET /?query=...
│   │   ├─ moz/
│   │   │   └─ metrics.ts          # Handle /moz-link-metrics requests
│   │   ├─ serp/
│   │   │   └─ google.ts           # Google SERP handler
│   │   ├─ api-keys.ts             # Handle /api/keys creation & usage
│   │   ├─ bing-serp.ts            # Bing SERP handler
│   │   ├─ youtube-serp.ts         # YouTube search handler
│   │   ├─ youtube-video-info.ts   # YouTube video info handler
│   │   ├─ html-source-code-search.ts # Handle domain HTML searches
│   │   ├─ enriched-domain-whois.ts    # Handle domain WHOIS + SEO metrics
│   │   └─ related-keywords.ts         # Handle related keywords retrieval
│   ├─ middleware/
│   │   ├─ auth.ts                 # Authentication checks
│   │   ├─ rate-limit.ts           # Rate limiting logic
│   │   ├─ error-handler.ts        # Error handling middleware
│   │   └─ request-logger.ts       # Request logging middleware
│   ├─ services/
│   │   ├─ clearbit.ts             # Integration with Clearbit
│   │   ├─ dataforseo.ts           # Integration with DataForSEO
│   │   ├─ logging.ts              # LoggingService to DB
│   │   └─ keywords-everywhere.ts  # Integration with KeywordsEverywhere (if used)
│   ├─ types/
│   │   ├─ common.ts               # Common interfaces (Env, SerpRequest, ApiResponse, etc.)
│   │   └─ dataforseo.ts           # DataForSEO-specific types (Google, Bing, YouTube, RelatedKeywords)
│   └─ index.ts                    # Entry point (Workers-compatible)
├── package.json                   # Project dependencies and scripts
├── README.md                      # Main developer documentation
└── wrangler.toml                  # Configuration for deploying to Cloudflare Workers
```

---

## Conclusion

This PRD gives a comprehensive, contextual overview of the Growth Marketing API Project. It outlines the architecture, enumerates primary features (SERP data retrieval, Clearbit integration, logging, rate limiting, related keywords discovery), and clarifies how the project’s documentation and file structure fit together. 

With the addition of new endpoints such as “related-keywords” and expanded existing functionalities (like “enriched-domain-whois” and “html-source-code-search”), the Growth Marketing API continues to evolve, providing more robust marketing and SEO tools. Contributors can navigate the codebase confidently—whether adding new search providers, implementing new endpoints, or refining existing features.
