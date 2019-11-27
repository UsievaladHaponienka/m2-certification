**6. Magento Code Generation** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/code-generation.html)

**Generated classes:**
- Factories. Factories are directly referenced within application code.
- Proxies. Proxies are directly referenced within dependency injection configuration.
- interceptors. Interceptors work behind the scenes and ARE NOT directly referenced in application code.

If the code generator implementation itself is changed, you must regenerate all the classes.

**Advantages of generated code:**
- The code is correct (if "basic" class is correct of cource)
- Code generation writes boilerplate code which decreases development time.
- All generated class of same type work same way
