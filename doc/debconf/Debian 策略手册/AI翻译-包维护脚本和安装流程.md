https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html

## 6. 包维护脚本和安装过程

### 6.1. 包维护脚本简介

可以提供一个包的一部分脚本，包管理系统将在你的包被安装、升级或移除时为你运行这些脚本。这些脚本是包元数据文件 `preinst`、`postinst`、`prerm` 和 `postrm`。它们必须是可执行文件；如果它们是脚本（推荐），则必须使用通常的 `#!` 约定开头。它们应该对任何人可读和可执行，并且绝不能是世界可写的。

包管理系统会查看这些脚本的退出状态。如果发生错误，它们必须以非零状态退出，这一点很重要，以便包管理系统可以停止其处理。对于 shell 脚本，这意味着你几乎总是需要使用 `set -e`（事实上，在编写 shell 脚本时通常都是如此）。当然，如果一切顺利，它们以零状态退出也很重要。

此外，在 `postinst` 脚本中使用 debconf 与用户交互的包，应安装一个 `config` 脚本作为包元数据文件。有关详细信息，请参阅 维护脚本中的提示。

当包升级时，在升级过程中会调用旧包和新包的脚本组合。如果你的脚本有任何复杂性，你需要注意这一点，并可能需要检查脚本的参数。广义上说，`preinst` 在包（的特定版本）解包之前调用，`postinst` 在其后调用；`prerm` 在包（的版本）被移除之前调用，`postrm` 在其后调用。

从维护脚本调用的程序通常不应带有前置路径。在安装开始之前，包管理系统会检查是否可以通过 `PATH` 环境变量找到程序 `ldconfig`、`start-stop-daemon` 和 `update-rc.d`。因此，这些程序以及任何预期会在 `PATH` 中的程序，都应该在不使用绝对路径名的情况下调用。维护脚本也不应重置 `PATH`，尽管它们可以选择通过前置或追加包特定目录来修改它。这些考虑实际上适用于所有 shell 脚本。

### 6.2. 维护脚本的幂等性

为了错误恢复过程，脚本必须是幂等的。这意味着如果它成功运行了一次，然后再次被调用，它不会出错或造成任何损害，而只是确保一切处于应有的状态。如果第一次调用失败或因某种原因中途中止，第二次调用应该只完成第一次未完成的事情（如果有的话），并且如果一切正常则以成功状态退出。[1]

### 6.3. 维护脚本的控制终端

不保证维护脚本在具有控制终端的情况下运行，并且可能无法与用户交互。如果没有可用的控制终端，它们必须能够回退到非交互行为。通过符合 Debian 配置管理规范的程序进行提示的维护脚本（参见 维护脚本中的提示）可以假定该程序将处理回退到非交互行为。

对于没有合理默认答案的高优先级提示，如果没有控制终端，维护脚本可以中止。但是，应尽可能避免这种情况，因为它会阻止自动化或无人值守安装。在大多数情况下，用户会认为这是包中的一个错误。

### 6.4. 退出状态

每个脚本必须返回零退出状态表示成功，或非零退出状态表示失败，因为包管理系统会查看这些脚本的退出状态，并根据该数据决定下一步要采取的操作。

### 6.5. 维护脚本调用方式摘要

接下来是维护脚本可能被调用的所有方式的摘要，以及这些脚本在调用时可以依赖哪些设施可用。以 `new-` 开头的脚本名称来自正在安装、升级或降级到的新版本包。以 `old-` 开头的脚本名称来自正在升级或降级的旧版本包。

**`preinst` 脚本可能以以下方式调用：**

*   `new-preinst install`
*   `new-preinst install old-version new-version`
*   `new-preinst upgrade old-version new-version`

    包尚未解包，因此 `preinst` 脚本不能依赖其包中包含的任何文件。只能假定基本包和预依赖项（`Pre-Depends`）可用。预依赖项至少已被配置过一次，但在调用 `preinst` 时，如果之前版本的预依赖项已被完全配置且自那以后未被移除，则它们可能只处于“已解包”或“半配置”状态。

*   `old-preinst abort-upgrade new-version`

    在解包新包后因 `postrm upgrade` 操作失败而升级出错时的错误处理期间调用。解包的文件可能部分来自新版本或部分丢失，因此脚本不能依赖包中包含的文件。包依赖项可能不可用。预依赖项至少是“已解包”，遵循上述相同规则，但如果预依赖项的升级失败，它们可能只是“半安装”状态。[2]

**`postinst` 脚本可能以以下方式调用：**

*   `postinst configure most-recently-configured-version`

    包中包含的文件将被解包。所有包依赖项至少是“已解包”。如果没有涉及循环依赖，所有包依赖项都将被配置。关于循环依赖情况下的行为，请参阅 元依赖中的讨论——依赖、推荐、建议、增强、预依赖。 中的讨论。

*   `old-postinst abort-upgrade new-version`
*   `conflictor's-postinst abort-remove in-favour package new-version`
*   `postinst abort-remove`
*   `deconfigured's-postinst abort-deconfigure in-favour failed-install-package version [ removing conflicting-package version ]`

    包中包含的文件将被解包。所有包依赖项至少是“半安装”状态，并且之前已被配置且未被移除。然而，在某些错误情况下，依赖项可能未被配置甚至未完全解包。[3] `postinst` 仍应尝试执行需要其依赖项的任何操作，因为它们通常可用，但如果这些操作失败，应考虑正确的错误处理方法。如果包依赖项中的命令或设施不可用，中止 `postinst` 操作通常是最佳方法。

**`prerm` 脚本可能以以下方式调用：**

*   `prerm remove`
*   `old-prerm upgrade new-version`
*   `conflictor's-prerm remove in-favour package new-version`
*   `deconfigured's-prerm deconfigure in-favour package-being-installed version [removing conflicting-package version]`

    被调用 `prerm` 的包将至少处于“半安装”状态。所有包依赖项至少是“半安装”状态，并且之前已被配置且未被移除。如果没有错误，所有依赖项至少是“已解包”，但这些操作可能在各种错误状态下被调用，其中依赖项由于部分升级而仅处于“半安装”状态。

*   `new-prerm failed-upgrade old-version new-version`

    当 `prerm upgrade` 失败时的错误处理期间调用。新包尚未解包，适用于 `preinst upgrade` 的所有相同约束都适用。

**`postrm` 脚本可能以以下方式调用：**

*   `postrm remove`
*   `postrm purge`
*   `old-postrm upgrade new-version`
*   `disappearer's-postrm disappear overwriter overwriter-version`

    `postrm` 脚本在包的文件已被移除或替换后调用。被调用 `postrm` 的包可能之前已被取消配置并且仅处于“已解包”状态，此时后续的包更改不再考虑其依赖项。因此，所有 `postrm` 操作只能依赖基本包，并且如果依赖项不可用，必须优雅地跳过任何需要包依赖项的操作。[4]

*   `new-postrm failed-upgrade old-version new-version`

    当旧的 `postrm upgrade` 操作失败时调用。新包将被解包，但只能依赖基本包和预依赖项。预依赖项要么已被配置，要么处于“已解包”或“半配置”状态，但之前已被配置且从未被移除。

*   `new-postrm abort-install`
*   `new-postrm abort-install old-version new-version`
*   `new-postrm abort-upgrade old-version new-version`

    在解包新包之前，作为 `preinst` 失败错误处理的一部分调用。可以假定与 `preinst` 可以假定的相同状态。

### 6.6. 安装或升级的解包阶段详情

安装/升级/覆盖/消失（即运行 `dpkg --unpack` 或 `dpkg --install` 的解包阶段）的过程如下。[5] 在每种情况下，如果发生重大错误（除非下面列出），操作通常会反向运行——这意味着维护脚本会以不同的参数按相反顺序被调用。这些就是下面列出的“错误回滚”调用。

1.  **通知当前已安装的包：**
    *   如果包的某个版本已经处于“已安装”状态，则调用：
        ```bash
        old-prerm upgrade new-version
        ```
    *   如果该脚本运行但以非零退出状态退出，`dpkg` 将尝试：
        ```bash
        new-prerm failed-upgrade old-version new-version
        ```
    *   如果此操作成功，升级继续。
    *   如果不成功，则错误回滚：
        ```bash
        old-postinst abort-upgrade new-version
        ```
    *   如果此操作成功，则旧版本处于“已安装”状态，否则旧版本处于“半配置”状态。

2.  **如果同时正在移除一个“冲突”包，或者任何包将被破坏（由于 `Breaks`）：**
    *   如果指定了 `--auto-deconfigure`，则为每个因 `Breaks` 而被取消配置的包调用：
        ```bash
        deconfigured's-prerm deconfigure in-favour package-being-installed version
        ```
        *错误回滚：*
        ```bash
        deconfigured's-postinst abort-deconfigure in-favour package-being-installed-but-failed version
        ```
        被取消配置的包被标记为需要配置，因此如果使用 `--install`，它们将在可能的情况下被重新配置。
    *   如果任何包依赖于正在被移除的冲突包并且指定了 `--auto-deconfigure`，则为每个这样的包调用：
        ```bash
        deconfigured's-prerm deconfigure in-favour package-being-installed version removing conflicting-package version
        ```
        *错误回滚：*
        ```bash
        deconfigured's-postinst abort-deconfigure in-favour package-being-installed-but-failed version removing conflicting-package version
        ```
        被取消配置的包被标记为需要配置，因此如果使用 `--install`，它们将在可能的情况下被重新配置。
    *   为准备移除每个冲突包，调用：
        ```bash
        conflictor's-prerm remove in-favour package new-version
        ```
        *错误回滚：*
        ```bash
        conflictor's-postinst abort-remove in-favour package new-version
        ```

3.  **运行新包的 `preinst`：**
    *   如果包正在被升级，调用：
        ```bash
        new-preinst upgrade old-version new-version
        ```
        *   如果失败，我们调用：
            ```bash
            new-postrm abort-upgrade old-version new-version
            ```
        *   如果成功，则调用 `old-postinst abort-upgrade new-version`。
        *   如果成功，则旧版本处于“已安装”状态，否则它被留在“已解包”状态。
        *   如果失败，则旧版本被留在“半安装”状态。
    *   否则，如果包安装了来自先前版本的一些配置文件（即，它处于“仅存配置文件”状态）：
        ```bash
        new-preinst install old-version new-version
        ```
        *错误回滚：*
        ```bash
        new-postrm abort-install old-version new-version
        ```
        *   如果失败，包被留在“半安装”状态，需要重新安装。
        *   如果成功，包被留在“仅存配置文件”状态。
    *   否则（即，包已被完全清除）：
        ```bash
        new-preinst install
        ```
        *错误回滚：*
        ```bash
        new-postrm abort-install
        ```
        *   如果错误回滚失败，包处于“半安装”阶段，需要重新安装。
        *   如果错误回滚成功，包处于“未安装”状态。

4.  新包的文件被解包，覆盖系统上可能已存在的任何文件，例如来自同一包的旧版本或来自另一个包的文件。旧文件的备份会暂时保存，如果出现任何问题，包管理系统将尝试在错误回滚期间将它们恢复。
    *   如果一个包包含的系统文件存在于另一个包中，则是一个错误，除非使用了 `Replaces`（参见 覆盖文件和替换包 - 替换）。
    *   如果一个包包含一个普通文件或其他类型的非目录，而另一个包在该位置有一个目录（同样，除非使用了 `Replaces`），则是一个更严重的错误。如果需要，可以使用 `--force-overwrite-dir` 覆盖此错误，但这是不可取的。相互覆盖文件的包会产生确定性的行为，但系统管理员难以理解。例如，如果一个包解包时覆盖了另一个包的文件，然后又被移除，很容易导致程序“丢失”。[6]
    *   目录永远不会被指向目录的符号链接替换，反之亦然；相反，将保留现有状态（是否为符号链接），如果有符号链接，`dpkg` 会跟随它。

5.  **如果包正在被升级：**
    *   调用：
        ```bash
        old-postrm upgrade new-version
        ```
    *   如果失败，`dpkg` 将尝试：
        ```bash
        new-postrm failed-upgrade old-version new-version
        ```
    *   如果成功，安装继续。
    *   如果不成功，错误回滚：
        ```bash
        old-preinst abort-upgrade new-version
        ```
        *   如果失败，旧版本被留在“半安装”状态。
        *   如果成功，`dpkg` 现在调用：
            ```bash
            new-postrm abort-upgrade old-version new-version
            ```
            *   如果失败，旧版本被留在“半安装”状态。
            *   如果成功，`dpkg` 现在调用：
                ```bash
                old-postinst abort-upgrade new-version
                ```
                *   如果失败，旧版本处于“已解包”状态。

6.  **这是不归点。** 如果 `dpkg` 到达这一步，在发生错误时它不会回退超过这一点。这将使包处于相当糟糕的状态，需要成功重新安装才能清理，但这是 `dpkg` 开始执行不可逆操作的时候。
    *   旧版本中存在但新版本中不存在的任何文件都将被移除。
    *   新的文件列表替换旧的。
    *   新的维护脚本替换旧的。

7.  **在安装期间，所有文件都被覆盖且不为依赖关系所需的任何包，被视为已被移除。** 对于每个这样的包，`dpkg` 调用：
    ```bash
    disappearer's-postrm disappear overwriter overwriter-version
    ```
    *   包的维护脚本被移除。
    *   它在状态数据库中被记录为处于正常状态，即“未安装”（它可能有的任何配置文件将被忽略，而不是由 `dpkg` 移除）。
    *   注意：消失的包不会调用它们的 `prerm`，因为 `dpkg` 事先不知道包会消失。

8.  我们正在解包的包中的任何也列在其他包文件列表中的文件，会从那些列表中移除。（如果存在“冲突”包，这将破坏其文件列表。）

9.  安装期间制作的备份文件被删除。

10. 新包的状态现在是正常的，并被记录为“已解包”。

11. **这是另一个不归点：** 如果冲突包的移除失败，我们不会回滚安装的其余部分。冲突包被留在半移除的中间状态。
    *   如果存在冲突包，我们继续执行移除操作（如下所述），从移除冲突包的文件开始（任何也在正在解包的包中的文件已经从冲突包的文件列表中移除，因此现在不会被移除）。

### 6.7. 配置详情

当我们配置一个包时（这发生在 `dpkg --install` 和 `dpkg --configure` 时），我们首先更新任何配置文件，然后调用：

```bash
postinst configure most-recently-configured-version
```

在配置期间发生错误后，不会尝试回滚。如果配置失败，包处于“半配置”状态，并生成错误消息。

如果没有最近配置的版本，`dpkg` 将传递一个空参数。[7]

### 6.8. 移除和/或配置清除详情

1.  ```bash
    prerm remove
    ```
    *   如果在因冲突而替换期间 `prerm` 失败：
        ```bash
        conflictor's-postinst abort-remove in-favour package new-version
        ```
    *   或者我们调用：
        ```bash
        postinst abort-remove
        ```
    *   如果失败，包处于“半配置”状态，否则保持“已安装”状态。

2.  包的文件被移除（配置文件除外）。

3.  ```bash
    postrm remove
    ```
    *   如果失败，没有错误回滚，包处于“半安装”状态。

4.  除 `postrm` 之外的所有维护脚本都被移除。

5.  如果我们不清除包，我们在此停止。注意，没有 `postrm` 且没有配置文件的包在移除时会自动被清除，因为除了 `dpkg` 状态外没有区别。

6.  配置文件和任何备份文件（`~` 文件、`#*#` 文件、`%` 文件、`.dpkg-{old,new,tmp}` 等）被移除。

7.  ```bash
    postrm purge
    ```
    *   如果失败，包保持“仅存配置文件”状态。

8.  包的文件列表被移除。

---

**脚注:**

[1] 这是为了在发生错误、用户中断 `dpkg` 或其他不可预见的情况发生时，当 `dpkg` 尝试重复该操作时，不会给用户留下一个严重损坏的包。

[2] 如果包的新版本不再预依赖于一个已被部分升级的包，则可能发生这种情况。

[3] 例如，假设包 `foo` 和 `bar` 处于“已安装”状态，且 `foo` 依赖于 `bar`。如果 `bar` 的升级开始后中止，然后尝试移除 `foo` 因其 `prerm` 脚本失败而失败，则 `foo` 的 `postinst abort-remove` 将在 `bar` 仅处于“半安装”状态时被调用。

[4] 这通常通过在调用之前检查 `postrm` 打算调用的命令或设施是否可用来完成。例如：`if [ "$1" = purge ] && [ -e /usr/share/debconf/confmodule ]; then . /usr/share/debconf/confmodule db_purge fi` 在 `postrm` 中，如果 debconf 已安装，则清除包的 debconf 配置。

[5] 部分问题是由于可以说是 `dpkg` 中的一个错误造成的。

[6] 这是 Debian 政策中关于文件覆盖的规则，旨在防止混乱和文件丢失。

[7] 历史注释：非常古老（1997 年前）的 `dpkg` 版本在这种情况下传递 `<unknown>`（包括尖括号）。更老的版本在任何情况下都不传递第二个参数。请注意，使用如此旧的 `dpkg` 版本进行升级可能由于其他原因而无法工作，即使你的 `postinst` 脚本处理了这种旧的参数行为。

**相关资源:**
*   维护者脚本流程图，我希望文本中的超链接保持
