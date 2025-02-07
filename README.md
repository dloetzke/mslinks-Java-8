About this fork
=======
This fork aims to make the upstream's code compilable with Java 8 and provide .jar releases for those who still work on Java 8, meaning they're no longer stuck on upstream's version 1.0.5.

It does not offer any support except for problems/changes related to Java 8 compilability. Please open issues and PRs on the upstream repository if they're not strictly related to the changes made in this fork.

**Tests have been disabled** as quite a few of them were failing while the fork compiles just fine and at least works for my purposes.

Releases on the Maven Central Repository are not planned.

mslinks
=======

### Summary
A library for parsing and creating Windows shortcut files (.lnk).

### Description
This is an implementation of [Shell Link (.LNK) Binary File Format](http://msdn.microsoft.com/en-us/library/dd871305.aspx) plus some reverse engineering. The library doesn't depend on any Windows-specific API, so it can be used in any environment.

This library allows opening existing .lnk files as well as creating new ones from scratch. You can set up many properties of the link such as working directory, command line arguments, icon, console text color, etc. The supported targets include:
* Files and directories on local filesystem with a drive letter as a root (absolute paths like `C:\path\to\target`)
* Files and directories in Samba shares (network paths like `\\host\share\path\to\target`)
* Files and directories in special Windows folders, such as Desktop, Documents, Downloads, etc. See available [GUIDs](https://github.com/DmitriiShamrikov/mslinks/wiki/GUIDs-table)

What is not supported (yet):
* Environment variables. They can be used in target path, but they are substituted at creation time, meaning the variables are taken from your (caller) environment, not the user's one
* Non-filesystem targets like Control Panel, Printers, etc.

The composition of classes reflect the data layout described in the format [specification](http://msdn.microsoft.com/en-us/library/dd871305.aspx), so it's recommended to take a look there if you are looking for something specific or want a detailed explanation of flags and constants. Otherwise, you can use the `ShellLinkHelper` class that provides methods for some general tasks.

### Examples
The easiest way to create a link with default parameters: `ShellLinkHelper.createLink("targetfile", "linkfile.lnk")`

The following example shows the creation of a link for a local .bat file and sets up a working directory, icon and console font. Note that the path has to be absolute and the drive letter has to be a separate parameter to enforce Windows-style paths. For simplicity, this example uses `java.nio.file.Path` to prepare the path assuming it runs on Windows. Take a look at [Examples.java](https://github.com/dloetzke/mslinks-Java-8/blob/master/test/mslinks/Examples.java) for a Linux compatible example.
```java
package mslinks;

import java.io.IOException;
import java.nio.file.Path;
import java.nio.file.Paths;

public class Main {
	public static void main(String[] args) throws IOException {
		ShellLink sl = new ShellLink()
			.setWorkingDir(Paths.get("..").toAbsolutePath().normalize().toString())
			.setIconLocation("%SystemRoot%\\system32\\SHELL32.dll");
		sl.getHeader().setIconIndex(128);
		sl.getConsoleData()
			.setFont(mslinks.extra.ConsoleData.Font.Consolas)
			.setFontSize(24)
			.setTextColor(5);

		Path targetPath = Paths.get("pause.bat").toAbsolutePath();
		String root = targetPath.getRoot().toString();
		String path = targetPath.subpath(0, targetPath.getNameCount()).toString();

		new ShellLinkHelper(sl)
			.setLocalTarget(root, path, Options.ForceTypeFile)
			.saveTo("testlink.lnk");
	}
}

```

The next example shows the creation of a link to a text file in a network share.
```java
package mslinks;

import java.io.IOException;

public class Main {
	public static void main(String[] args) throws IOException {
		ShellLinkHelper link = new ShellLinkHelper(new ShellLink());
		link.setNetworkTarget("\\\\host\\share\\testfile.txt", Options.ForceTypeFile);
		link.saveTo("testlink.lnk");
	}
}

```

The next example shows the usage of special folders. This allows setting a path relative to a system-known location which can be different for different users. See a [list](https://github.com/DmitriiShamrikov/mslinks/wiki/GUIDs-table) of available folders.
```java
package mslinks;

import java.io.IOException;

public class Main {
	public static void main(String[] args) throws IOException {
		ShellLinkHelper link = new ShellLinkHelper(new ShellLink());
		link.setSpecialFolderTarget(Registry.CLSID_DOCUMENTS, "pause.bat", Options.ForceTypeFile);
		link.saveTo("testlink.lnk");
	}
}

```

### Download
* [releases page](https://github.com/dloetzke/mslinks-Java-8/releases)
