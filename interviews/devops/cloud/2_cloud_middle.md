# Cloud — Middle

## Вопросы

- [AWS S3 + CloudFront: как настроить для фронтенда?](#aws-s3--cloudfront-как-настроить-для-фронтенда)
- [Что такое Edge Computing?](#что-такое-edge-computing)
- [Как настроить custom domain и SSL?](#как-настроить-custom-domain-и-ssl)
- [Что такое serverless functions?](#что-такое-serverless-functions)

---

## AWS S3 + CloudFront: как настроить для фронтенда?

S3 хранит статику, CloudFront — CDN поверх. Настройка: S3 bucket без публичного доступа, CloudFront OAC (Origin Access Control). Cache: long TTL для иммутабельных ресурсов (JS/CSS с hash), short TTL для index.html.

---

## Что такое Edge Computing?

Edge — выполнение кода на серверах CDN рядом с пользователем. Cloudflare Workers, Vercel Edge Functions, AWS Lambda@Edge. Кейсы: A/B тестирование, персонализация, аутентификация, геолокация — без запроса к origin.

---

## Как настроить custom domain и SSL?

DNS: CNAME запись → CDN. SSL/TLS: Let's Encrypt автоматически через платформу. HTTPS redirect: `301` HTTP → HTTPS. HSTS: `Strict-Transport-Security: max-age=31536000; includeSubDomains`.

---

## Что такое serverless functions?

Serverless (FaaS) — функции без управления сервером. AWS Lambda, Vercel Functions, Netlify Functions. Автоматическое масштабирование, оплата за вызов. Next.js API Routes в Vercel — serverless functions.
