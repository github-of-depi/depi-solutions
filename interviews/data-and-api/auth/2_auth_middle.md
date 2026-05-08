# Auth — Middle

## Вопросы

- [Как реализовать refresh token flow?](#как-реализовать-refresh-token-flow)
- [Что такое OAuth2 Authorization Code + PKCE?](#что-такое-oauth2-authorization-code--pkce)
- [Как настроить NextAuth.js?](#как-настроить-nextauthjs)
- [Что такое SameSite cookie атрибут?](#что-такое-samesite-cookie-атрибут)
- [Как защититься от CSRF?](#как-защититься-от-csrf)
- [Что такое OpenID Connect?](#что-такое-openid-connect)

---

## Как реализовать refresh token flow?

Access token (короткоживущий, ~15мин) + refresh token (долгоживущий, httpOnly cookie). При 401: axios interceptor автоматически запрашивает новый access token через refresh endpoint, повторяет оригинальный запрос.

```typescript
axiosInstance.interceptors.response.use(
  res => res,
  async error => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      await refreshToken(); // обновить access token
      return axiosInstance(error.config);
    }
    return Promise.reject(error);
  }
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое OAuth2 Authorization Code + PKCE?

PKCE (Proof Key for Code Exchange) — расширение OAuth2 для публичных клиентов (SPA, mobile). Клиент генерирует `code_verifier` (случайная строка), отправляет `code_challenge` (SHA256 от verifier). Сервер проверяет verifier при обмене кода на токен. Защита от перехвата authorization code.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как настроить NextAuth.js?

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth";
import GoogleProvider from "next-auth/providers/google";

const handler = NextAuth({
  providers: [
    GoogleProvider({ clientId: process.env.GOOGLE_ID!, clientSecret: process.env.GOOGLE_SECRET! }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) token.role = user.role;
      return token;
    },
    async session({ session, token }) {
      session.user.role = token.role;
      return session;
    },
  },
});
export { handler as GET, handler as POST };
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое SameSite cookie атрибут?

`SameSite=Strict` — cookie не отправляется при cross-site запросах. `SameSite=Lax` — отправляется при навигации (GET), не при субресурсах (форма POST на другой домен). `SameSite=None; Secure` — всегда (нужен для OAuth redirect). Lax — дефолт современных браузеров.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как защититься от CSRF?

CSRF — злоумышленник заставляет браузер отправить запрос с cookie жертвы. Защита: SameSite=Strict/Lax cookie, CSRF-токены (Double Submit Cookie), проверка Origin/Referer заголовков. JWT в Authorization header (не cookie) — иммунитет к CSRF.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое OpenID Connect?

OIDC — слой идентификации поверх OAuth2. Добавляет ID token (JWT с данными пользователя: sub, email, name). `userinfo` endpoint для получения профиля. «Войти через Google» — это OIDC, а не просто OAuth2.

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
