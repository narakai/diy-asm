## 主体流程

## 目录结构
```
master\
    doc\    ;一些参考文档
    i18n\   ;多语言支持
    pics\   ;图标与演示
    resources\
        hoverinfo.md  ;悬浮提示
        instructions-reference.json ; 文档链接
    samples\ ;简单实例
    src\
    syntaxes\ ;语法高亮
    tools\    ;汇编工具
        dosbox\
        js-dos\
        masm\
        player\
        tasm\
    language-configuration.json 语言配置
    package.json
    snippets.json
    webpack.config.js
```

## package.json
```json
{
    "activationEvents": [
        "onCommand:masm-tasm.dosboxhere",
        "onLanguage:assembly",
        "onLanguage:tasm",
        "onLanguage:masm"
    ],
    "contributes": {
        "languages": [
            {
                "id": "assembly",
                "aliases": [
                "assembly(DOS)"
                ],
                "extensions": [
                ".asm",
                ".inc"
                ],
                "configuration": "./language-configuration.json"
            }
        ],
        "snippets": [
            {
                "language": "assembly",
                "path": "./snippets.json"
            }
        ],
        "grammars": [
            {
                "language": "assembly",
                "scopeName": "source.asm",
                "path": "./syntaxes/assembly.tmLanguage.json"
            }
        ],
        // 右键菜单栏 menus.editor/context
        "menus": {
            "editor/context": [
                {
                "when": "resourceExtname == .ASM || resourceExtname == .asm && editorFocus",
                "command": "masm-tasm.openEmulator",
                "group": "0_MASM-TASM@1"
                },
                {
                "when": "resourceExtname == .ASM || resourceExtname == .asm && editorFocus",
                "command": "masm-tasm.runASM",
                "group": "0_MASM-TASM@2"
                },
                {
                "when": "resourceExtname == .ASM || resourceExtname == .asm && editorFocus",
                "command": "masm-tasm.debugASM",
                "group": "0_MASM-TASM@3"
                }
            ]
        },
        // 命令注册 使用多语言nls
        "commands": [
            {
                "command": "masm-tasm.openEmulator",
                "title": "%editor.openEmu%"
            },
            {
                "command": "masm-tasm.runASM",
                "title": "%editor.runAsm%"
            },
            {
                "command": "masm-tasm.debugASM",
                "title": "%editor.debugAsm%"
            },
            {
                "command": "masm-tasm.cleanalldiagnose",
                "title": "%command.cleanalldianose%"
            },
            {
                "command": "masm-tasm.dosboxhere",
                "title": "%command.dosboxhere%"
            }
        ],
        // config 
        "configuration": {}
    }
}
```
### package.json的config
```json
{
"configuration": {
      "type": "object",
      "title": "MASM/TASM",
       // 支持的字段
       /*
       {
           masmtasm.ASM.toolspath:"",
           masmtasm.ASM.MASMorTASM:"",
           masmtasm.ASM.emulator:"",
           masmtasm.ASM.separateSpace:"",
           masmtasm.ASM.clean:"",
           masmtasm.ASM.savefirst:"",
           masmtasm.language.programmaticFeatures:"",
           masmtasm.language.Hover:"",
           masmtasm.dosbox.config:"",
           masmtasm.dosbox.command:"",
           masmtasm.dosbox.console:"",
           masmtasm.jsdos.viewColumn:"",
           masmtasm.jsdos.wdosbox:"",
           masmtasm.dosbox.more:{},
           masmtasm.msdos.more:{}
       }
       */   
      "properties": {
        "masmtasm.ASM.toolspath": {
          "type": "string",
          "markdownDescription": "%config.toolspath.mddescription%"
        },
        "masmtasm.ASM.MASMorTASM": {
          "type": "string",
          "default": "TASM",
          "description": "%config.masmortasm.description%",
          "enum": [
            "TASM",
            "MASM"
          ],
          "enumDescriptions": [
            "%config.masmortasm.enum1%",
            "%config.masmortasm.enum2%"
          ]
        },
        "masmtasm.ASM.emulator": {
          "type": "string",
          "default": "dosbox",
          "description": "%config.emulator.description%",
          "enum": [
            "jsdos",
            "dosbox",
            "msdos player",
            "auto"
          ],
          "enumDescriptions": [
            "%config.emulator.jsdos%",
            "%config.emulator.dosbox%",
            "%config.emulator.player%",
            "%config.emulator.auto%"
          ]
        },
        "masmtasm.ASM.separateSpace": {
          "type": "boolean",
          "default": false,
          "markdownDescription": "%config.separateSpace%"
        },
        "masmtasm.ASM.clean": {
          "type": "boolean",
          "default": true,
          "description": "%config.clean%"
        },
        "masmtasm.ASM.savefirst": {
          "type": "boolean",
          "default": true,
          "description": "%config.savefirst%"
        },
        "masmtasm.language.programmaticFeatures": {
          "type": "boolean",
          "default": true,
          "description": "%config.PLF%"
        },
        "masmtasm.language.Hover": {
          "type": "boolean",
          "default": true,
          "description": "%config.hover%"
        },
        "masmtasm.dosbox.run": {
          "type": "string",
          "default": "choose",
          "description": "%config.boxrun.description%",
          "enum": [
            "keep",
            "exit",
            "pause",
            "choose"
          ],
          "enumDescriptions": [
            "%config.boxrun.enum1%",
            "%config.boxrun.enum2%",
            "%config.boxrun.enum3%",
            "%config.boxrun.choose%"
          ]
        },
        "masmtasm.dosbox.config": {
          "type": "object",
          "additionalProperties": {
            "type": "string"
          },
          "default": {
            "SDL.windowresolution": "1024x640",
            "SDL.output": "opengl"
          },
          "markdownDescription": "%config.boxconfig.description%"
        },
        "masmtasm.dosbox.command": {
          "type": "string",
          "description": "%config.boxcommand%"
        },
        "masmtasm.dosbox.console": {
          "type": "string",
          "default": "redirect",
          "description": "%config.boxconsole.description%",
          "enum": [
            "min",
            "normal",
            "noconsole",
            "redirect(show)",
            "redirect"
          ],
          "markdownEnumDescriptions": [
            "%config.boxconsole.min%",
            "%config.boxconsole.normal%",
            "%config.boxconsole.noconsole%",
            "%config.boxconsole.redirect-show%",
            "%config.boxconsole.redirect%"
          ]
        },
        "masmtasm.jsdos.viewColumn": {
          "markdownDescription": "%config.jsdos.viewColumn%",
          "type": "number",
          "default": -1,
          "enum": [
            -1,
            -2
          ],
          "enumDescriptions": [
            "%config.jsdos.viewColumn.Active%",
            "%config.jsdos.viewColumn.Besides%"
          ]
        },
        "masmtasm.jsdos.wdosbox": {
          "markdownDescription": "The extension packaged the wdosbox.js as default variant. We can use different variant but make sure console programs/shell is usable. See [js-dos-622-faq-changing-dosbox-variant](https://js-dos.com/#js-dos-622-faq-changing-dosbox-variant)",
          "type": "string"
        },
        "masmtasm.dosbox.more": {
          "markdownDescription": "%config.dosbox.more%",
          "default": {
            "open": [
              "set path=c:\\masm;c:\\tasm"
            ],
            "masm": [
              "set path=c:\\masm",
              "masm ${filename}.ASM; >X:\\ASM.LOG ",
              "@type X:\\ASM.LOG",
              "if exist ${filename}.OBJ link ${filename}.OBJ; >X:\\LINK.LOG ",
              "@type X:\\LINK.LOG"
            ],
            "tasm": [
              "set path=c:\\tasm",
              "tasm /zi ${filename}.ASM >X:\\ASM.LOG ",
              "@type X:\\ASM.LOG",
              "if exist ${filename}.OBJ tlink /v/3 ${filename}.obj >X:\\LINK.LOG ",
              "@type X:\\LINK.LOG"
            ],
            "masm_debug": [
              "if exist ${filename}.exe c:\\masm\\debug ${filename}.exe"
            ],
            "tasm_debug": [
              "if exist ${filename}.exe c:\\tasm\\TD ${filename}.exe"
            ],
            "run": [
              "@if not exist ${filename}.exe exit ",
              "${filename}.exe",
              "@echo (END)Here is the end of the program's output"
            ],
            "after_action": [
              "@choice Do you need to keep the DOSBox",
              "@IF ERRORLEVEL 2 exit",
              "@IF ERRORLEVEL 1 echo on"
            ]
          },
          "properties": {
            "open": {
              "description": "command exec when open dosbox",
              "type": "array"
            },
            "masm": {
              "description": "The commands to exec for assembling and linking in dosbox when using masm",
              "type": "array"
            },
            "tasm": {
              "description": "The commands to exec for assembling and linking in dosbox when using tasm",
              "type": "array"
            },
            "tasm_debug": {
              "description": "The commands to exec for debuging in dosbox when using tasm",
              "type:": "array"
            },
            "masm_debug": {
              "description": "The commands to exec for debuging in dosbox when using masm",
              "type:": "array"
            },
            "run": {
              "description": "The commands to run the generated file",
              "type": "array"
            },
            "after_action": {
              "description": "The commands to exec after run or debug\n when set `masmtasm.dosbox.run` as `choose`",
              "type": "array"
            }
          }
        },
        "masmtasm.jsdos.more": {
          "markdownDescription": "%config.jsdos.more%",
          "default": {
            "open": [
              "set path=c:\\asm\\masm;c:\\asm\\tasm",
              "cd code"
            ],
            "masm": [
              "masm ${filename}.ASM;",
              "if exist ${filename}.OBJ link ${filename}.OBJ;"
            ],
            "tasm": [
              "tasm /zi ${filename}.ASM",
              "if exist ${filename}.OBJ tlink /v/3 ${filename}.obj"
            ],
            "masm_debug": [
              "if exist ${filename}.exe debug ${filename}.exe"
            ],
            "tasm_debug": [
              "if exist ${filename}.exe TD ${filename}.exe"
            ],
            "run": [
              "if exist ${filename}.exe ${filename}.exe"
            ]
          },
          "properties": {
            "open": {
              "description": "The commands to exec when open JSdos webview",
              "type": "array"
            },
            "masm": {
              "description": "The commands to exec for assembling and linking in Wdosbox when using masm",
              "type": "array"
            },
            "tasm": {
              "description": "The commands to exec for assembling and linking in Wdosbox when using tasm",
              "type": "array"
            },
            "tasm_debug": {
              "description": "The commands to exec for debuging in Wdosbox when using tasm",
              "type:": "array"
            },
            "masm_debug": {
              "description": "The commands to exec for debuging in Wdosbox when using masm",
              "type:": "array"
            },
            "run": {
              "description": "The commands to run the generated file",
              "type": "array"
            }
          }
        },
        "masmtasm.msdos.more": {
          "description": "%config.msdos.more%",
          "default": {
            "workspace": "c:\\.dosasm",
            "path": "${toolpath}\\player;${toolpath}\\masm;${toolpath}\\tasm;c:\\.dosasm\\tasm;c:\\.dosasm\\masm;",
            "masm": "player\\playerasm.bat \"${toolpath}\" MASM \"${fullname}\"",
            "tasm": "player\\playerasm.bat \"${toolpath}\" TASM \"${fullname}\"",
            "masm_debug": "msdos -v5.0 debug \"${filename}.exe\"",
            "tasm_debug": "msdos TD \"${filename}.exe\"",
            "run": "msdos \"${filename}\""
          },
          "properties": {
            "workspace": {
              "description": "The workspace when using separate space",
              "type": "string"
            },
            "path": {
              "description": "The path needed to add to the environment value `PATH`",
              "type": "string"
            },
            "masm": {
              "description": "The commands to exec for assembling and linking in cmd when using masm",
              "type": "string"
            },
            "tasm": {
              "description": "The commands to exec for assembling and linking in cmd when using tasm",
              "type": "string"
            },
            "tasm_debug": {
              "description": "(Currently not support)The commands to exec for debuging in cmd when using tasm",
              "type:": "string"
            },
            "masm_debug": {
              "description": "The commands to exec for debuging in cmd when using masm",
              "type:": "string"
            },
            "run": {
              "description": "The command to run the generated file",
              "type": "string"
            }
          }
        }
      }
    },
    "problemMatchers": [
      {
        "owner": "MASM",
        "fileLocation": "autoDetect",
        "pattern": [
          {
            "regexp": "^\\s*(.*)\\((\\d+)\\):\\s+(error|warning)\\s+([A-Z]\\d+:\\s+.*)$",
            "file": 1,
            "line": 2,
            "severity": 3,
            "message": 4,
            "loop": true
          }
        ]
      },
      {
        "owner": "TASM",
        "fileLocation": "autoDetect",
        "pattern": [
          {
            "regexp": "^\\s*\\*+(Error|Warning)\\*+\\s+(.*)\\((\\d+)\\)\\s+(.*)$",
            "line": 3,
            "severity": 1,
            "message": 4,
            "file": 2,
            "loop": true
          }
        ]
      }
    ]
}
}
```
### package.json的多语言实现
```json
{
	"editor.openEmu": "Open Emulator",
	"editor.runAsm": "Run ASM code",
	"editor.debugAsm": "Debug ASM code",
	"command.cleanalldianose": "MASM/TASM: Clean all diagnose information generated by the extension",
	"command.dosboxhere": "DOSBox here: Open DosBox and prepare the environment",
	"config.toolspath.mddescription": "the path of your tools folder for assembly, used to replace the built-in one",
	"config.masmortasm.description": "use TASM or MASM to operate your assembly codes",
	"config.masmortasm.enum1": "use MASM toolset, including masm.exe,link.exe,debug.exe...",
	"config.masmortasm.enum2": "use TASM toolset, including tasm.exe,tlink.exe,td.exe...",
	"config.emulator.description": "DOS environment emulator",
	"config.emulator.jsdos": "Use jsdos(wdosbox), run in webview",
	"config.emulator.dosbox": "Use DOSBox(most stable),need to install dosbox by yourself",
	"config.emulator.player": "Use MSDOS-player for most cases(use dosbox for TD),this may be more quiet",
	"config.emulator.auto": "use MSDOS-player to compile,use dosbox to run，use dosbox for TD.exe,MSDOS player for debug.exe",
	"config.savefirst": "Save the file before Open dosbox, run and debug ASM codes",
	"config.boxrun.description": "What to do after run code in dosbox",
	"config.boxrun.enum1": "do nothing, manually input exit or click 'x' or press 'Ctrl+F9' to exit",
	"config.boxrun.enum2": "exit DOSBox automatically",
	"config.boxrun.enum3": "pause and then exit",
	"config.boxrun.choose": "use choose command to decide keep dosbox or not",
	"config.boxcommand": "If defined, the extension will use this command to open dosbox instead of default command",
	"config.boxconsole.description": "what to do with the console window",
	"config.boxconsole.min": "- For windows, use command like `start/min dosbox`, this will **Minimize** the console window\n- for other OS, will not process the console information",
	"config.boxconsole.normal": "- For windows, use command like `dosbox`, this will **Create** a console window by dosbox\n- for other OS, will not process the console information",
	"config.boxconsole.noconsole": "don't show console\n\n- For windows, use command like `dosbox -noconsole` to redirect the console stdout and stderr to files\n- for other OS, will not process the console information",
	"config.boxconsole.redirect-show": "Redirect the console stdout and stderr to VSCode output channel, and show. For windows it will not display in real time",
	"config.boxconsole.redirect": "Redirect the console stdout and stderr to VSCode output channel. For windows it will not display in real time",
	"config.boxconfig.description": "dosbox configurations that will be used, see [link](https://www.dosbox.com/wiki/Dosbox.conf),use property like `\"AUTOEXEC\":\"echo hi\\ndir\"` if need to use *AUTOEXEC*",
	"config.hover": "Display Hover information or not, restart VSCode to apply",
	"config.PLF": "Experimental programmatic language features like outline,jump to definition/reference. Restart needed",
	"config.separateSpace": "Use a Separate space to keep the workspace clean. This will cause problem with multi-file assembly. \n\n- The extension will copy source code file to this space\n- When the emulator's fileSystem do not support your file path, this will be enabled by default",
	"config.clean": "clean related files before action",
	"config.jsdos.viewColumn": "the view Column where the jsdos webview will create. See [ViewColumn](https://code.visualstudio.com/api/references/vscode-api#ViewColumn)",
	"config.jsdos.viewColumn.Active": "Active panel",
	"config.jsdos.viewColumn.Besides": "Besides panel",
	"config.dosbox.more": "The commands to use when in dosbox mode.\n\n- The extension will mount tools folder to dosbox's disk C, the file's folder to disk D\n- a separate space as dosbox's disk X,the extension will read file X:\\ASM.LOG to process assembler's message\n",
	"config.jsdos.more": "The commands to use when in wdosbox(jsdos mode)",
	"config.msdos.more": "The commands to use when in msdos player or auto mode"
}
```

