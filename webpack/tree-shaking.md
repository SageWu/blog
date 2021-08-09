# Tree Shaking

## 介绍

用于删除`dead-code`，从而减小打包后文件的大小。依赖于`es6-module`，因为该模块系统是静态化的，可以在编译阶段得知模块与模块之间的依赖关系以及模块的导出。

## Modules-Level

作用于整个模块。相关配置为`optimization.sideEffects`，在`production`模式下，默认为true。\
可以将package.json文件中的`sideEffects`属性设置为false，指明导入模块是不会产生其他副作用的，webpack可以安全的进行tree-shaking。如果没有设置或者设置为true，则webpack会进行分析，再决定是否移除未使用的模块。

## Statements-Level

作用于模块内的语句。相关配置为`optimization.usedExports`，`optimization.minimize`，在`production`模式下，默认为true。

## 具体过程

> 1、获取`entry`配置，生成`dependency`，解析配置的path，得到绝对路径。
> 2、创建`module`。使用配置的`loader`对其内容进行处理，结果解析为AST，收集依赖。递归处理依赖，静态依赖存放在`dependencies`列表中，动态依赖则存放在`blocks`列表中。边的类型有`HarmonyImportSideEffectDependency`、`HarmonyImportSpecifierDependency`、`AsyncDependenciesBlock`。
> 3、最终构建完成模块依赖图。
> 4、确定每个模块依赖的`usedExports`，根据`HarmonyImportSpecifierDependency`。移除只存在`HarmonyImportSideEffectDependency`边的模块，只要导入模块是不会产生其他副作用的。
> 5、构建`chunk`，动态依赖单独成`chunk`。构建`chunk`的内容。
> `terser`对`chunk`的内容进行处理：删除未使用代码，压缩，混淆。`terser`也会生成AST对代码进行分析，删除为使用代码。
