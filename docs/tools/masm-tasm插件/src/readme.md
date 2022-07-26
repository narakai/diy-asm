## 主流程
- main.ts
- runCode.ts
- DOSBox.ts
## 目录
```
src\
    asm ; 汇编与模拟器的操作
    emulator ; 模拟器接口
    lanugage ; 语法功能
```

## extension.ts
```ts
import * as vscode from 'vscode';
import * as nls from 'vscode-nls';


import { provider } from './language/provider';
import { AsmCommands } from './ASM/main';

// 多语言初始化
nls.config({ messageFormat: nls.MessageFormat.bundle, bundleFormat: nls.BundleFormat.standalone })();
const localize: nls.LocalizeFunc = nls.loadMessageBundle();


export function activate(context: vscode.ExtensionContext): void {
	console.log(localize("activate.hello", 'Congratulations, your extension "masm-tasm" is now active!'));
	//provide programmaic language features like hover,references,outline(symbol)
    // 注册语法功能
	provider(context);
	//run and debug the code in dosbox or msdos-player by TASM ot MASM
    // 注册dosbox,msdos-player功能
	AsmCommands(context);
}

export function deactivate(): void {
	console.log('extension deactivated');
}

```