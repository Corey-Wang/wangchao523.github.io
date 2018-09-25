---
title: vscode extensions install
date: 2018-08-21 11:23:50
categories: 技术 vscode
tags:
---
## 背景
最近在搞一个vscode插件，但由于公司内容使用，无法发不到市场，所以必须自己搞一个升级能力

## 命令行安装插件
```
code --install-extension vsix_file
```
如果提示没有code命令，需要在在vscode中执行 cmd+shift+p --> Shell Command: Install 'code' command in PATH

## vscode安装插件源码分析（通过vsix包安装）
https://github.com/Microsoft/vscode/blob/master/src/vs/platform/extensionManagement/node/extensionManagementService.ts
```
install(zipPath: string): TPromise<void> {
    zipPath = path.resolve(zipPath);

    return validateLocalExtension(zipPath)
        .then(manifest => {
            const identifier = { id: getLocalExtensionIdFromManifest(manifest) };
            if (manifest.engines && manifest.engines.vscode && !isEngineValid(manifest.engines.vscode)) {
                return TPromise.wrapError<void>(new Error(nls.localize('incompatible', "Unable to install Extension '{0}' as it is not compatible with Code '{1}'.", identifier.id, pkg.version)));
            }
            return this.removeIfExists(identifier.id)
                .then(
                    () => this.checkOutdated(manifest)
                        .then(validated => {
                            if (validated) {
                                this.logService.info('Installing the extension:', identifier.id);
                                this._onInstallExtension.fire({ identifier, zipPath });
                                return this.getMetadata(getGalleryExtensionId(manifest.publisher, manifest.name))
                                    .then(
                                        metadata => this.installFromZipPath(identifier, zipPath, metadata, manifest),
                                        error => this.installFromZipPath(identifier, zipPath, null, manifest))
                                    .then(
                                        () => { this.logService.info('Successfully installed the extension:', identifier.id); },
                                        e => {
                                            this.logService.error('Failed to install the extension:', identifier.id, e.message);
                                            return TPromise.wrapError(e);
                                        });
                            }
                            return null;
                        }),
                    e => TPromise.wrapError(new Error(nls.localize('restartCode', "Please restart Code before reinstalling {0}.", manifest.displayName || manifest.name))));
        });
}

```

## 我的实现
```
 /**
     * 插件升级
     */
    public async extensionUpdate() {
        console.log('coding extension is updating...');
        const ext = vscode.extensions.getExtension('coding');
        if (ext !== undefined) {
            const json = ext.packageJSON;
            request(`${SERVER_URL_BASE}versionUpdate?name=${json.id}&v=${json.version}`, function (error, response, body) {
                if (response && response.statusCode === 200) {
                    const localDir = `${os.homedir}/.coding/vsix/`;
                    
                    if (!fs.existsSync(localDir)) {
                        fs.mkdirSync(localDir);
                    }
                    const updateJson = JSON.parse(body);
                    download(updateJson.vsix, localDir, {filename: 'coding-new.vsix'}).then(() => {
                        const vsix = `${localDir}coding-new.vsix`;
                        cp.exec(`bin/code --install-extension ${vsix}`, {cwd: vscode.env.appRoot}, (error, stdout, stderr) => {
                            if (error) {
                                console.error(`exec error: ${error}`);
                                return;
                            } else if (stderr) {
                                console.error(`stderr: ${stderr}`);
                                return;
                            } else if (stdout.indexOf('successfully installed') < 0) {
                                console.log(`stdout: ${stdout}`);
                                return;
                            }
                            console.log(`stdout: ${stdout}`);
                            fs.unlinkSync(vsix);        // delete file
                            vscode.window.showInformationMessage('Successfully installed the coding extension. Reload to enable it.', 'Reload Now').then(selected => {
                                if (selected === 'Reload Now') {
                                    vscode.commands.executeCommand('workbench.action.reloadWindow');
                                }
                            });
                            console.log('coding of extension update successed!');
                            
                        });
                    });
                } else {
                    console.log('update check error response code: ', response);
                }
            });
        }
    }
```