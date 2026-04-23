function Abbreviate-Path {
    param(
        [string]$path = $PWD.Path,
        [int]$n_max_dirs = 4
    )

    $path = $path.TrimEnd('\', '/')
    $path = $path -replace '^([^\\/:]+\\)?FileSystem::', ''
    $path = $path -replace '^[^\\/:]+\\(?=\w+::)', ''
    $home_dir = $HOME
    if ($home_dir -and $path.Length -ge $home_dir.Length -and
        [string]::Equals($path.Substring(0, $home_dir.Length), $home_dir, [System.StringComparison]::OrdinalIgnoreCase) -and
        ($path.Length -eq $home_dir.Length -or $path[$home_dir.Length] -eq '\' -or $path[$home_dir.Length] -eq '/')) {
        $root = '~' + [System.IO.Path]::DirectorySeparatorChar
        $rest = if ($path.Length -eq $home_dir.Length) { '' } else { $path.Substring($home_dir.Length + 1) }
    } else {
        $root = [System.IO.Path]::GetPathRoot($path)
        $rest = $path.Substring($root.Length)
    }
    $dirs = $rest.Split([char[]]@('\', '/'), [System.StringSplitOptions]::RemoveEmptyEntries)

    if ($dirs.Count -le $n_max_dirs) {
        return ($root + $rest).TrimEnd('\', '/')
    }

    $keepEnd   = [int][Math]::Ceiling($n_max_dirs / 2)
    $keepStart = $n_max_dirs - $keepEnd

    $startDirs = if ($keepStart -gt 0) { $dirs[0..($keepStart - 1)] } else { @() }
    $endDirs   = if ($keepEnd   -gt 0) { $dirs[($dirs.Count - $keepEnd)..($dirs.Count - 1)] } else { @() }

    $sep   = [System.IO.Path]::DirectorySeparatorChar
    $parts = @($startDirs) + '...' + @($endDirs)
    return ($root + ($parts -join $sep)).TrimEnd('\', '/')
}
