# git 工具

## 检索仓库所有分支内文件是否包含某个字符

```shell
// windows 终端执行环境
git fetch --all

$branches = git branch -r | Where-Object { $_ -notmatch '->' }
$matchedFiles = @{}

foreach ($branch in $branches) {
    $branchName = $branch.Trim()
    Write-Host "Checking branch: $branchName"
    
    $files = git ls-tree -r --name-only $branchName

    foreach ($file in $files) {
        try {
            $content = git show "${branchName}:${file}" 2>$null
            if ($content -match 'keyword') {
                $key = "${branchName}:${file}"
                $matchedFiles[$key] = $true
            }
        } catch {
            # 忽略错误
        }
    }
}

$matchedFiles.Keys | Sort-Object | Set-Content matched-content-files.txt
```

## 查找文件名包含 keyword 的文件

```shell
git fetch --all

$branches = git branch -r | Where-Object { $_ -notmatch '->' }
$allFiles = @{}

foreach ($branch in $branches) {
    $files = git ls-tree -r --name-only $branch.Trim()
    foreach ($file in $files) {
        $allFiles[$file] = $true
    }
}

$allFiles.Keys | Where-Object { $_ -like "*zhongxin*" } | Sort-Object | Set-Content matched-files.txt

```
