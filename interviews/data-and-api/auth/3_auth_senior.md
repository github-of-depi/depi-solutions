# Auth — Senior

## Вопросы

- [Как реализовать silent refresh?](#как-реализовать-silent-refresh)
- [Как обеспечить logout на всех вкладках?](#как-обеспечить-logout-на-всех-вкладках)
- [Что такое token binding и Demonstrating Proof of Possession (DPoP)?](#что-такое-token-binding-и-demonstrating-proof-of-possession-dpop)
- [Как строить role-based access control (RBAC) на фронтенде?](#как-строить-role-based-access-control-rbac-на-фронтенде)
- [Как защититься от XSS в контексте аутентификации?](#как-защититься-от-xss-в-контексте-аутентификации)

---

## Как реализовать silent refresh?

Скрытый `<iframe>` или background запрос к authorization server для обновления токенов без взаимодействия пользователя. Более современный подход: refresh token rotation + background fetch до истечения access token.

---

## Как обеспечить logout на всех вкладках?

`BroadcastChannel API` или `localStorage` событие (StorageEvent) — broadcast logout события между вкладками. При получении — очистить state, редирект на login.

```typescript
const channel = new BroadcastChannel("auth");
function logout() {
  clearTokens();
  channel.postMessage({ type: "logout" });
  router.push("/login");
}
channel.onmessage = (e) => {
  if (e.data.type === "logout") { clearTokens(); router.push("/login"); }
};
```

---

## Что такое token binding и Demonstrating Proof of Possession (DPoP)?

DPoP — расширение OAuth2, привязывающее токен к конкретному клиенту через криптографическую пару ключей. Даже перехваченный токен бесполезен без приватного ключа клиента. Используется в высокосекурных приложениях (банки, eGov).

---

## Как строить role-based access control (RBAC) на фронтенде?

Роли из JWT token claims. Кастомные хуки + higher-order components для проверки прав.

```typescript
function usePermission(permission: string): boolean {
  const { user } = useAuth();
  return user?.permissions.includes(permission) ?? false;
}

function ProtectedButton({ permission, ...props }: Props) {
  const canAccess = usePermission(permission);
  return canAccess ? <button {...props} /> : null;
}
```

---

## Как защититься от XSS в контексте аутентификации?

XSS + localStorage = кража токенов. Меры: Content Security Policy (CSP) ограничивает выполнение скриптов, httpOnly cookies для токенов, sanitization входных данных, Trusted Types API (Chrome). Использование фреймворков (React) — защита от innerHTML XSS.
