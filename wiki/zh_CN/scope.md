# 管理依赖的作用域

InversifyJS 默认使用 `transient` 作用域，但您也可以使用 `singleton` 或 `request` 作用域：

```ts
container.bind<Shuriken>("Shuriken").to(Shuriken).inTransientScope(); // Default
container.bind<Shuriken>("Shuriken").to(Shuriken).inSingletonScope();
container.bind<Shuriken>("Shuriken").to(Shuriken).inRequestScope();
```

## 关于 `inSingletonScope`

有很多可用的绑定类型：

```ts
interface BindingToSyntax<T> {
    to(constructor: { new (...args: any[]): T; }): BindingInWhenOnSyntax<T>;
    toSelf(): BindingInWhenOnSyntax<T>;
    toConstantValue(value: T): BindingWhenOnSyntax<T>;
    toDynamicValue(func: (context: Context) => T): BindingWhenOnSyntax<T>;
    toConstructor<T2>(constructor: Newable<T2>): BindingWhenOnSyntax<T>;
    toFactory<T2>(factory: FactoryCreator<T2>): BindingWhenOnSyntax<T>;
    toFunction(func: T): BindingWhenOnSyntax<T>;
    toAutoFactory<T2>(serviceIdentifier: ServiceIdentifier<T2>): BindingWhenOnSyntax<T>;
    toProvider<T2>(provider: ProviderCreator<T2>): BindingWhenOnSyntax<T>;
}
```

根据作用域的表现我们可以将这些类型的绑定分为两个大组：

- 注入一个 `object` 的绑定
- 注入一个 `function` 的绑定

### 注入一个 `object` 的绑定

在这个组中包含以下绑定类型：

```ts
interface BindingToSyntax<T> {
    to(constructor: { new (...args: any[]): T; }): BindingInWhenOnSyntax<T>;
    toSelf(): BindingInWhenOnSyntax<T>;
    toConstantValue(value: T): BindingWhenOnSyntax<T>;
    toDynamicValue(func: (context: Context) => T): BindingInWhenOnSyntax<T>;
}
```

默认使用 `inTransientScope`，我们可以为这些绑定类型，选择作用域。但是除了 `toConstantValue`，因为它始终只使用 `inSingletonScope`。

当我们使用 `to`, `toSelf` 或 `toDynamicValue` 第一次调用 `container.get`，InversifyJS 容器会尝试产生一个对象实例，使用一个构造方法的值或工厂动态值。
如果作用域已经设置为 `inSingletonScope`， 值会被缓存。
第二次调用 `container.get` 会得到相同的资源ID， 如果已经选择了 `inSingletonScope`， InversifyJS会尝试从缓存中获取值。

Note that a class can have some dependencies and a dynamic value can access other types via the current context. These dependencies may or may not be a singleton independently of the selected scope of their parent object in their respective composition tree,
注意，一个类可以有一些依赖项，和一个通过当前上下文访问其他类型的动态值。

### Bindings that will inject a `function`

In this group are included the following types of binding:

```ts
interface BindingToSyntax<T> {
    toConstructor<T2>(constructor: Newable<T2>): BindingWhenOnSyntax<T>;
    toFactory<T2>(factory: FactoryCreator<T2>): BindingWhenOnSyntax<T>;
    toFunction(func: T): BindingWhenOnSyntax<T>;
    toAutoFactory<T2>(serviceIdentifier: ServiceIdentifier<T2>): BindingWhenOnSyntax<T>;
    toProvider<T2>(provider: ProviderCreator<T2>): BindingWhenOnSyntax<T>;
}
```

We cannot select the scope of this types of binding because the value to be injected (a factory `function`) is always a singleton. However, the factory internal implementation may or may not return a singleton.

For example, the following binding will inject a factory which will always be a singleton.

```ts
container.bind<interfaces.Factory<Katana>>("Factory<Katana>").toAutoFactory<Katana>("Katana");
```

However, the value returned by the factory may or not be a singleton:

```ts
container.bind<Katana>("Katana").to(Katana).inTransientScope();
// or
container.bind<Katana>("Katana").to(Katana).inSingletonScope();
```

## About `inRequestScope`

When we use inRequestScope we are using an special kind of singleton.

- The `inSingletonScope` creates a singleton that will last for the entire life cycle of a type binding. This means that the `inSingletonScope` can be cleared up from memory when we unbind a type binding using `container.unbind`.

- The `inRequestScope` creates a singleton that will last for the entire life cycle of one call to the `contaner.get`, `container.getTagged` or `container.getNamed` methods. Each call to one of this methods will resolve a root dependency and all its sub-dependencies. Internally, a dependency graph known as the "resolution plan" is created by InversifyJS. The `inRequestScope` scope will use one single instance for objects that appear multiple times in the resolution plan. This reduces the number of required resolutions and it can be used as a performance optimization in some cases.
