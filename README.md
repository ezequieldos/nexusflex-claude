# Nexus Flex para Claude

Plugin oficial de [Nexus Flex](https://nexusflex.com.ar): trae **juntos** el MCP de Nexus Flex (altas, consultas y control por foto; nunca mueve dinero) y la skill de **control por foto**.

## Instalar

**Claude Desktop / claude.ai:** Personalizar → Plugins → Agregar → Agregar marketplace → escribí `ezequieldos/nexusflex-claude` → Sincronizar → en la lista aparece **nexusflex** → Agregar.

**Claude Code:**

```bash
claude plugin marketplace add ezequieldos/nexusflex-claude
claude plugin install nexusflex@nexusflex-claude
```

Necesitás Node.js 20 o más nuevo. La primera vez que lo usás te muestra un link y un código para **autorizar** con tu cuenta de Nexus Flex (no se pone la contraseña en ningún lado).

El código del MCP es el paquete npm [`nexusflex-mcp`](https://www.npmjs.com/package/nexusflex-mcp); este repositorio solo tiene el marketplace, la configuración del plugin y las skills. Se genera desde el repo principal: no editar a mano.
