## DosBox.new()
```ts
export class DOSBox extends dosboxCore implements EMURUN {
    // 配置处理
    constructor(conf: Config) {
        const vscConf = new BoxVSCodeConfig();
        let boxconsole: WINCONSOLEOPTION | undefined = undefined;
        if (vscConf.command === undefined || vscConf.command.length === 0) {
            switch (vscConf.console) {
                case "min":
                    boxconsole = WINCONSOLEOPTION.min;
                    break;
                case "normal":
                    boxconsole = WINCONSOLEOPTION.normal;
                    break;
                case "noconsole":
                case "redirect(show)":
                case "redirect":
                default:
                    boxconsole = WINCONSOLEOPTION.noconsole;
                    break;
            }
        }
        super(conf.Uris.dosbox.fsPath, vscConf.command, boxconsole);
        this.forceCopy = false;
        this._conf = conf;
        this.vscConfig = vscConf;

        this._BOXrun = vscConf.run;

        if (this.redirect) {
            this.stdoutHander = (message: string, text: string, code: number): void => {
                Logger.send({
                    title: localize('dosbox.console.stdout', '[dosbox console stdout] No.{0}', code.toString()),
                    content: message
                });
                if (vscConf.console === 'redirect(show)') {
                    Logger.OutChannel.show(true);
                }
                else if (vscConf.console === 'redirect(hide)') {
                    Logger.OutChannel.hide();
                }
            };
            this.stderrHander = (message: string, _text: string, code: number): void => {
                Logger.send({
                    title: localize('dosbox.console.stderr', '[dosbox console stderr] No.{0}', code.toString()),
                    content: message
                });
            };
        }
    }
}
```

## DosBox.prepare
- 生成配置文件`dosbox.config`
```ts
export class DOSBox extends dosboxCore implements EMURUN {
    async prepare(opt?: ASMPREPARATION): Promise<boolean> {
        //write the config file for extension
        this.confFile = Uri.joinPath(this._conf.Uris.globalStorage, DOSBOX_CONF_FILENAME);
        await writeBoxconfig(this.confFile, this.vscConfig.config);
        if (opt) {
            this.forceCopy = !opt.src.dosboxFsReadable;
        };
        this.vscConfig.replacer = (val: string): string => settingsStrReplacer(val, this._conf, opt ? opt.src : undefined);
        return true;
    }
}
```
## DosBox.openEmu
- 
```ts
export class DOSBox extends dosboxCore implements EMURUN {
    openEmu(folder: Uri): Promise<unknown> {
        return this.runDosbox(folder, []);
    }
}
```

## DoxBos.runDosbox
```ts
export class DOSBox extends dosboxCore implements EMURUN {
    public async runDosbox(folder: Uri, more?: string[], opt?: { exitwords: boolean }): Promise<DOSBoxStd> {
        // 获取open的配置 生成box命令参数
        const boxcmd: string[] = this.vscConfig.getAction('open');
        boxcmd.push(
            `@mount c \\\"${this._conf.Uris.tools.fsPath}\\\"`,//mount the tools folder as disk C
            `@mount d \\\"${folder.fsPath}\\\"`,//mount the folder of source file as disk D
            `@mount X \\\"${this._conf.Uris.globalStorage.fsPath}\\\"`,//mount a separate space as X for the extension to read logs
            "d:"//switch to the disk of source code file
        );
        if (more) { boxcmd.push(...more); }
        if (opt?.exitwords) { boxcmd.push(...this.boxruncmd); }
        //Logger.log(boxcmd);
        if (boxcmd.length > DOSBOX_CMDS_LIMIT) {
            const omit = boxcmd.slice(DOSBOX_CMDS_LIMIT - 1);
            const unit8 = new TextEncoder().encode(omit.join('\n'));
            const dst = Uri.joinPath(this._conf.Uris.globalStorage, 'more.bat');
            await fs.writeFile(dst, unit8);
            boxcmd.splice(DOSBOX_CMDS_LIMIT - 1, omit.length);
            boxcmd.push('@x:\\more.bat');
        }
        // 启动执行
        return this.run(boxcmd);
    }
}
```
## DoxBos.run
```ts
export class DOSBox extends dosboxCore implements EMURUN {
    public run(boxcmd: string[], opt?: DOSBoxOption): Promise<DOSBoxStd> {
        const preOpen = opt?.preOpen ? opt?.preOpen : "";
        const param = [];
        if (this.console === WINCONSOLEOPTION.noconsole) {
            param.push('-noconsole');
        }
        if ((typeof opt?.confFile) === 'string') {
            param.push(`-conf "${opt?.confFile}"`);
        }
        else if (opt?.confFile === undefined) {
            param.push(`-conf "${this.confFile?.fsPath}"`);
        }
        if (opt?.param && opt?.param.length > 0) {
            param.push(...opt.param);
        }
        if (boxcmd.length > 0) {
            const mapper = (val: string): string => `-c "${val}"`;
            param.push(...boxcmd.map(mapper));
        }
        // runViaChildProcess
        return this.runViaChildProcess(preOpen + this._core + ' ' + param.join(" "));
    }
}
```

## DoxBox.runViaChildProcess
```ts
export class DOSBox extends dosboxCore implements EMURUN {t
    private runViaChildProcess(command: string, ignoreWinStd?: boolean): Promise<DOSBoxStd> {
        //console.log(command);
        this._count++;
        const output: DOSBoxStd = {
            flag: BoxStdNOTE.normal,
            stdout: "",
            stderr: '',
            exitcode: undefined
        };
        return new Promise(
            (resolve, reject) => {
                // exec
                const child = exec(
                    command,
                    {
                        cwd: this._cwd,
                    },
                    (error, stdout, stderr): void => {
                        if (error) {
                            reject(error);
                        }
                        else if (process.platform === 'win32' && this.redirect && this._cwd && ignoreWinStd !== true) {
                            // 读取命令行
                            winReadConsole(this._cwd).then(
                                (value) => {
                                    if (value) {
                                        output.stderr = value.stderr;
                                        output.stdout = value.stdout;
                                        resolve(output);
                                        this.stderrHander(value.stderr, value.stderr, this._count);
                                        this.stdoutHander(value.stdout, value.stdout, this._count);
                                    }
                                });
                        }
                        else {
                            output.stdout = stdout;
                            output.stderr = stderr;
                            if (process.platform === 'win32') {
                                if (this.redirect === false) {
                                    output.flag = BoxStdNOTE.winNonoconsole;
                                }
                                else if (this._cwd === undefined) {
                                    output.flag = BoxStdNOTE.noCwd;
                                }
                                // else if (ignoreWinStd === false) {
                                //     output.flag = BoxStdNOTE.winCancelled;
                                // }
                                else if (ignoreWinStd === true) {
                                    output.flag = BoxStdNOTE.winCancelledByUser;
                                }

                            }
                            resolve(output);
                        }
                    });

                child.on('exit', (code) => {
                    output.exitcode = code;
                    if (code !== 0) {
                        let msg = `Open dosbox Failed with exitcode ${code} `;
                        msg += 'executing shell command:' + command;
                        window.showErrorMessage(msg);
                    }
                });
                if (this.redirect && process.platform !== 'win32') {
                    child.stdout?.on('data', (data) => {
                        this._stdout += data;
                        this.stdoutHander(data, this._stdout, this._count);
                    });
                    child.stderr?.on('data', (data) => {
                        this._stderr += data;
                        this.stderrHander(data, this._stderr, this._count);
                    });
                }
            }
        );
    }
}
```

## DoxBos.winReadConsole
```ts
async function winReadConsole(folder: string): Promise<{ stdout: string; stderr: string }> {
    const cwd = Uri.file(folder);
    const fs = workspace.fs;
    const dirs = await fs.readDirectory(cwd);
    const output = { stdout: "", stderr: "" };
    let file = inDirectory(dirs, ['stderr.txt', FileType.File]);
    if (file) {
        output.stderr = (await fs.readFile(Uri.joinPath(cwd, file[0]))).toString();
    }
    file = inDirectory(dirs, ['stdout.txt', FileType.File]);
    if (file) {
        output.stdout = (await fs.readFile(Uri.joinPath(cwd, file[0]))).toString();
    }
    return output;
}
```