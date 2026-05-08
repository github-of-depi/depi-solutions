# Cloud — Expert

## Вопросы

- [Как строить Infrastructure as Code для фронтенда?](#как-строить-infrastructure-as-code-для-фронтенда)
- [Как работать с WebAssembly на Edge?](#как-работать-с-webassembly-на-edge)

---

## Как строить Infrastructure as Code для фронтенда?

Terraform / Pulumi / CDK: вся инфраструктура (S3, CloudFront, Route53, ACM) как код. Преимущества: версионирование, review, reproducibility, disaster recovery.

```typescript
// Pulumi — TypeScript IaC
import * as aws from "@pulumi/aws";
const bucket = new aws.s3.Bucket("frontend", { website: { indexDocument: "index.html" } });
const cdn = new aws.cloudfront.Distribution("cdn", { origins: [{ domainName: bucket.bucketRegionalDomainName }], ... });
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как работать с WebAssembly на Edge?

Cloudflare Workers поддерживает Wasm модули. Rust/C++ → Wasm → запуск на Edge. Кейсы: криптографические операции, resize изображений, обработка данных без origin. Fastly Compute@Edge — Wasm-first platform.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
