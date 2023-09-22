# Lombok

## 注解列表

- `@Getter`
- `@Setter`
- `@ToString`
- `@EqualsAndHashCode`
- `@Builder`
  - `@Builder.Default` 保留Bean字段的默认值
- `@NoArgsConstructor`
- `@RequiredArgsConstructor`
- `@AllArgsConstructor`
- `@Data` 相当于`@Getter` `@Setter` `@RequiredArgsConstructor` `@ToString` `@EqualsAndHashCode`的组合

## slf4j使用

```java
@Slf4j
public class LogExample {
}
```

```java
public class LogExample {
    private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);
}
```

## mapstruct使用
