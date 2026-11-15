# Spring Security: основы и практическая модель

## Аутентификация и авторизация

Аутентификация отвечает на вопрос «кто пользователь?». Авторизация — «что ему разрешено?». Успешный вход не означает доступ ко всем операциям.

## Security Filter Chain

В servlet-приложении Spring Security использует цепочку фильтров до controller. Фильтры могут извлекать credentials, устанавливать SecurityContext и проверять доступ.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .build();
    }
}
```

Это учебный пример. В реальном приложении отдельно оцениваю CSRF, CORS, способ хранения credentials и HTTPS. Я использую современную конфигурацию через `SecurityFilterChain`, а не устаревшие примеры с `WebSecurityConfigurerAdapter`.

## UserDetails и PasswordEncoder

`UserDetails` описывает пользователя для Spring Security: username, password hash, authorities и флаги состояния. `UserDetailsService` загружает пользователя.

Пароли нельзя хранить открытым текстом. `PasswordEncoder` проверяет введённый пароль относительно сохранённого hash. Это не обратимое шифрование: пароль не расшифровывается обратно. Использую поддерживаемый алгоритм password hashing, например BCrypt или Argon2.

## Roles и authorities

Authority — конкретное право. Role — распространённая договорённость, обычно authority с префиксом `ROLE_`. `hasRole("ADMIN")` обычно проверяет `ROLE_ADMIN`.

## CSRF и CORS

CSRF — атака с нежелательным запросом, в котором браузер автоматически прикладывает credentials. CORS — браузерная политика доступа к cross-origin ответам. CORS не является аутентификацией и не защищает API от небраузерных клиентов.

Я не отключаю CSRF просто потому, что API REST. Сначала определяю, используются ли cookies и browser-based credentials.

## Session и stateless

При session-based authentication сервер хранит состояние сессии. При stateless bearer-token подходе сервер проверяет токен на запросах без серверной HTTP-сессии. У обоих подходов есть компромиссы.

## JWT

JWT — формат токена с claims и подписью. Подписанный JWT обычно не зашифрован: его содержимое можно прочитать. Секреты туда не помещаю.

Проверяю подпись, допустимый алгоритм, issuer, audience, срок действия, назначение токена, правила обновления и отзыва. Для проверки использую поддерживаемые библиотеки и механизмы Spring Security OAuth2 Resource Server, а не самописную криптографию.

## Method Security

`@EnableMethodSecurity` включает проверки на методах. `@PreAuthorize` проверяет выражение до вызова метода.

```java
@PreAuthorize("hasAuthority('task:delete')")
public void deleteTask(long id) {
    // Бизнес-операция.
}
```

## Практика

1. Закрываю все endpoint по умолчанию.
2. Открываю только `/public/**`.
3. Добавляю пользователя и PasswordEncoder.
4. Разделяю USER и ADMIN.
5. Пишу тесты на 401, 403 и разрешённый доступ.
6. Изучаю OAuth2 Resource Server и JWT по официальному руководству.
