## AsmAction.new
```ts
export class AsmAction implements Disposable {
    constructor(context: ExtensionContext) {
        this.ctx = context;
        // 配置初始化 configration.md
        this._config = new Config(context);

        this.landiag = new AssemblerDiag();
    }
    // 跟配置读取不同的模拟器
    static getEmulator(emu: DOSEMU, conf: Config): EMURUN {
        switch (emu) {
            case DOSEMU.dosbox:
                // DOSBox()模拟器
                // ./emulator/DOSBox.md
                return new DOSBox(conf);
            case DOSEMU.auto:
                return new AutoMode(conf);
            case DOSEMU.msdos:
                return new MsdosPlayer(conf);
            case DOSEMU.jsdos:
                return new JSDos(conf);
            default:
                window.showWarningMessage('use dosbox as emulator');
                return new DOSBox(conf);
        }
    }
    
}

```
## runcode.runcode
```ts
export class AsmAction implements Disposable {
    public async runcode(command: ASMCMD, uri?: Uri): Promise<RUNCODEINFO> {
        // 初始化模拟器
        const emulator = AsmAction.getEmulator(this._config.DOSemu, this._config);
        // 
        const output: RUNCODEINFO = { diagCode: DIAGCODE.null };
        //get the target file
        // 获得源代码
        let src: SRCFILE | undefined;
        if (uri) {
            src = new SRCFILE(uri);
        }
        else if (window.activeTextEditor?.document) {
            src = new SRCFILE(window.activeTextEditor.document.uri);
        }
        //construct the source code file class
        if (!src) {
            window.showErrorMessage('no source file specified');
        }
        else if (!await emulator.prepare({ src: src, act: command })) {
            console.warn(this._config.DOSemu + ' emulator is not ready');
        }
        else {
            // 打开文件
            const doc = await workspace.openTextDocument(src.uri);
            if (doc.isDirty && this._config.savefirst) {
                await doc.save();
            }
            //output the message of the command
            const msg = { title: "", content: src.pathMessage() };
            switch (command) {
                case ASMCMD.OpenEmu:
                    msg.title = localize("openemu.msg", "\n[execute]Open emulator and prepare environment");
                    break;
                case ASMCMD.run:
                    msg.title = localize("run.msg", "\n[execute]use {0} in {1} to Run ASM code file:", this._config.MASMorTASM, this._config.DOSemu);
                    break;
                case ASMCMD.debug:
                    msg.title = localize("debug.msg", "\n[execute]use {0} in {1} to Debug ASM code file:", this._config.MASMorTASM, this._config.DOSemu);
                    break;
            }
            if (this._config.Separate || emulator.forceCopy) {
                const dst = emulator.copyUri === undefined ? this._config.Uris.workspace : emulator.copyUri;
                await src.copyto(dst);
                msg.content += localize('copy.msg', `\ncopied as "{0}"`, src.uri.fsPath);
            }
            if (this._config.Clean) {
                await src.cleanDir();
            }
            Logger.send(msg);

            const msgProcessor: MSGProcessor =
                (message: string | { asm: string; link: string }, opt?: { preventWarn: boolean }) => {
                    const msg = typeof (message) === 'string' ? message : message.asm;
                    output.message = message;
                    const diag = this.landiag.ErrMsgProcess(msg, doc, this.ASM);
                    output.diagnose = diag;
                    if (diag?.code !== undefined) { output.diagCode = diag?.code; }
                    if (diag) {
                        Logger.send({
                            title: localize("diag.msg", "[assembler's message] {0} Error,{1}  Warning collected", diag.error.toString(), diag.warn),
                            content: msg
                        });
                    }
                    switch (diag?.code) {
                        case DIAGCODE.ok:
                            return true;
                        case DIAGCODE.hasWarn:
                            if (opt?.preventWarn) {
                                return false;
                            }
                            else {
                                return this.showWarnInfo();
                            }
                        case DIAGCODE.hasError:
                            this.showErrorInfo();
                            Logger.OutChannel.show(true);
                            return false;
                    }
                    return false;
                };
            // 执行命令
            switch (command) {
                case ASMCMD.OpenEmu:
                    // 打开Dos环境
                    output.emulator = emulator.openEmu(src.folder);
                    break;
                case ASMCMD.run:
                    // 运行代码 
                    // emulator/DOSBox.md##Run()
                    output.emulator = await emulator.Run(src, msgProcessor);
                    break;
                case ASMCMD.debug:
                    // 调试代码 
                    // emulator/DOSBox.md##Debug()
                    output.emulator = await emulator.Debug(src, msgProcessor);
                    break;
            };
        }
        return output;
    }
}
```
## BoxHere
```ts
export class AsmAction implements Disposable {
    // masm-tasm.dosboxhere
    public async BoxHere(uri?: Uri, emulator?: DOSEMU): Promise<unknown> {
        // 文件夹
        let folder: Uri | undefined = undefined;
        if (uri) {
            folder = uri;
        }
        else {
            if (window.activeTextEditor?.document) {
                folder = Uri.joinPath(window.activeTextEditor?.document.uri, '../');
            }
            else if (workspace.workspaceFolders) {
                if (workspace.workspaceFolders.length === 1) {
                    folder = workspace.workspaceFolders[0].uri;
                }
                else if (workspace.workspaceFolders.length > 1) {
                    const a = await window.showWorkspaceFolderPick();
                    if (a) { folder = a.uri; }
                }
            }
        }

        //choose the emulator
        // 根据配置获取模拟器
        const dosemu = emulator ? emulator : DOSEMU.dosbox;
        const emu = AsmAction.getEmulator(dosemu, this._config);
        //打开模拟器
        // emu.prepare()
        if (folder && await emu.prepare()) {
            // emu.openEmu()
            const output = await emu.openEmu(folder);
            return output;
        }
        else {
            window.showWarningMessage('no folder to open \nThe extension use the activeEditor file\'s folder or workspace folder');
        }
    }
}
```