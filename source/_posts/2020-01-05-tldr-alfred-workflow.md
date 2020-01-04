---
title: tldr alfred workflow
date: 2020-01-05 01:02:45
tags:

 - alfred
 - tech
 - workflow
---

### tldr

>Too Long; Didn’t Read

偶然间,看到一个比较好用的工具.可以看做是简化版本的man.这个是官网的简介:

> A collection of simplified and community-driven man pages.

举个例子,看看tar这个命令的对比

这个是`man tar`的部分内容

```bash
TAR(1)                    BSD General Commands Manual                   TAR(1)

NAME
     tar -- manipulate tape archives

SYNOPSIS
     tar [bundled-flags <args>] [<file> | <pattern> ...]
     tar {-c} [options] [files | directories]
     tar {-r | -u} -f archive-file [options] [files | directories]
     tar {-t | -x} [options] [patterns]

DESCRIPTION
     tar creates and manipulates streaming archive files.  This implementation can extract from tar, pax, cpio, zip, jar, ar, xar, rpm, 7-zip,
     and ISO 9660 cdrom images and can create tar, pax, cpio, ar, zip, 7-zip, and shar archives.

     The first synopsis form shows a ``bundled'' option word.  This usage is provided for compatibility with historical implementations.  See
     COMPATIBILITY below for details.

     The other synopsis forms show the preferred usage.  The first option to tar is a mode indicator from the following list:
     -c      Create a new archive containing the specified items.  The long option form is --create.
     -r      Like -c, but new entries are appended to the archive.  Note that this only works on uncompressed archives stored in regular files.
             The -f option is required.  The long option form is --append.
     -t      List archive contents to stdout.  The long option form is --list.
     -u      Like -r, but new entries are added only if they have a modification date newer than the corresponding entry in the archive.  Note
             that this only works on uncompressed archives stored in regular files.  The -f option is required.  The long form is --update.
```

再来看看tldr的版本

```markdown
# tar

> Archiving utility.
> Often combined with a compression method, such as gzip or bzip.
> More information: <https://www.gnu.org/software/tar>.

- Create an archive from files:

`tar cf {{target.tar}} {{file1}} {{file2}} {{file3}}`

- Create a gzipped archive:

`tar czf {{target.tar.gz}} {{file1}} {{file2}} {{file3}}`

- Extract a (compressed) archive into the current directory:

`tar xf {{source.tar[.gz|.bz2|.xz]}}`

- Extract an archive into a target directory:

`tar xf {{source.tar}} -C {{directory}}`

- Create a compressed archive, using archive suffix to determine the compression program:

`tar caf {{target.tar.xz}} {{file1}} {{file2}} {{file3}}`

- List the contents of a tar file:

`tar tvf {{source.tar}}`

- Extract files matching a pattern:

`tar xf {{source.tar}} --wildcards {{"*.html"}}
```

显然tldr的版本要简单明了的多,这么好的功能当然要和Alfred这个神器结合一下.

### Alfred Workflow

基本的思路就是把tldr的git仓库clone到本地,然后写个接口来嫁接Alfred的数据格式要求.直接上代码:

```java
package name.chengchao.springrun.service.alfredplugin;

import java.io.File;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ConcurrentHashMap;

import org.apache.commons.io.FileUtils;
import org.apache.commons.lang.StringUtils;

import name.chengchao.springrun.model.AlfredLine;

public class TLDRService {

	private final static String PATH = "/root/tldr/pages/common";
	private static List<String> commandList = new ArrayList<String>();
	private static ConcurrentHashMap<String, List<AlfredLine>> cache = new ConcurrentHashMap<String, List<AlfredLine>>();

	static {
		init();
	}

	@SuppressWarnings("unchecked")
	public static void init() {
		commandList.clear();
		List<File> files = (List<File>) FileUtils.listFiles(new File(PATH), new String[] { "md" }, false);
		for (File file : files) {
			commandList.add(StringUtils.substringBefore(file.getName(), "."));
		}
	}

	@SuppressWarnings("unchecked")
	public static List<AlfredLine> search(String action, String keyword) throws Exception {
		keyword = StringUtils.trim(keyword);
		List<AlfredLine> list = new ArrayList<>();
		int count = 0;
		boolean bingo = false;
		for (String command : commandList) {
			if (command.equalsIgnoreCase(keyword)) {
				bingo = true;
				break;
			}
			if (command.startsWith(keyword)) {
				count++;
				AlfredLine alfredLine = new AlfredLine();
				alfredLine.setTitle(command);
				alfredLine.setSubtitle("");
				alfredLine.setArg("");
				alfredLine.setAutocomplete(command);
				list.add(alfredLine);
				if (count >= 10) {
					break;
				}
			}
		}

		// 命中一个具体命令的逻辑
		if (bingo) {
			if (cache.get(keyword) != null) {
				return cache.get(keyword);
			}

			list.clear();
			File file = new File(PATH + "/" + keyword + ".md");
			if (file.exists()) {
				List<String> lines = FileUtils.readLines(file);
				AlfredLine alfredLine = null;
				for (String line : lines) {
					if (line.startsWith("-")) {
						if (alfredLine != null) {
							list.add(alfredLine);
						}
						alfredLine = new AlfredLine();
						alfredLine.setSubtitle(line.substring(1, line.length() - 1));
					} else if (line.startsWith("`") && alfredLine != null) {
						alfredLine.setTitle(line.substring(1, line.length() - 1));
						alfredLine.setArg(line.substring(1, line.length() - 1));
					}

				}
				if (alfredLine != null) {
					list.add(alfredLine);
				}

				cache.put(keyword, list);
			}
		}
		return list;
	}
}
```

看看效果

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/alfred_tldr_1.png" style="zoom:50%;" />

<img src="https://chengchaosite.oss-cn-hangzhou.aliyuncs.com/resource-container/image/alfred_tldr_2.png" style="zoom:50%;" />