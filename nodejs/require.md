# require

## require.resolve

通过该方法解析决定最终加载的文件路径。

> require(x)\
> 如果x为核心模块，则直接返回。\
> 如果为绝对路径或者相对路径，则按照路径进行查找。
>> 如果为文件，解析顺序是x, x.js, x.json, x.node。\
>> 如果为目录，则检查是否存在package.json，并使用main字段。\
>> 以上还未找到，则检查index文件。
>
> 如果为加载自己所属的库，x以package.json的name字段为前缀，根据exports字段决定。\
> 如果为第三方模块，则从加载模块的位置往上，查找node_modules目录，并尝试在其中寻找模块。
>> x为name和subpath的组合，并且该node_modules目录存在name/package.json。根据exports字段以及subpath决定文件路径。\
>> 在该node_modules目录下，按文件方式解析。\
>> 在该node_modules目录下，按照目录方式解析。

条件性导出\
在package.json的exports字段中配置

* node：匹配nodejs环境
* node-addons：匹配nodejs环境，更明确表明该入口依赖c++ addon
* import：通过import或import()导入模块
* require：通过require导入模块
* default：作为fallback
