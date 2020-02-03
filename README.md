# coc-java-debug

An experimental [extension for coc.nvim](https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions) to enable the
[Java Debug Server](https://github.com/Microsoft/java-debug) extension for the [jdt.ls](https://github.com/eclipse/eclipse.jdt.ls) language server as loaded by [coc-java](https://github.com/neoclide/coc-java).

#### Disclaimer

*This being experimental, I can not make any guarantees about this plugin or example config working for your setup.*

### Prerequisites

Be sure to have the [coc-java](https://github.com/neoclide/coc-java#quick-start) extension installed.

### Install

    :CocInstall https://github.com/dansomething/coc-java-debug

### Uninstall

    :CocUninstall coc-java-debug

## Example usage with [Vimspector](https://puremourning.github.io/vimspector-web/)

This example will use Vimspector as the user interface for interacting with the [Java Debug Server](https://github.com/Microsoft/java-debug) from within Vim.

It will demonstrate attaching to a Java program that is running with remote debugging enabled.

#### Setup Vimspector

Install the [Vimspector](https://github.com/puremourning/vimspector#installation) plugin for Vim.

Add a `.vimspector.json` file in the root directory of your Java project with the following contents.

    {
      "adapters": {
        "java-debug-server": {
          "name": "vscode-java",
          "port": "${AdapterPort}"
        }
      },
      "configurations": {
        "Java Attach": {
          "adapter": "java-debug-server",
          "configuration": {
            "request": "attach",
            "host": "127.0.0.1",
            "port": "5005"
          },
          "breakpoints": {
            "exception": {
              "caught": "N",
              "uncaught": "N"
            }
          }
        }
      }
    }

*Review the [Vimspector config](https://puremourning.github.io/vimspector/configuration.html) docs for what's possible with this file.*

#### Configure Vim

Add the following config to your `~/.vimrc` file or wherever appropriate for your Vim setup.

    function! JavaStartDebugCallback(err, port)
      execute "cexpr! 'Java debug started on port: " . a:port . "'"
      call vimspector#LaunchWithSettings({ "configuration": "Java Attach", "AdapterPort": a:port })
    endfunction

    function JavaStartDebug()
      call CocActionAsync('runCommand', 'vscode.java.startDebugSession', function('JavaStartDebugCallback'))
    endfunction

    nmap <F1> :call JavaStartDebug()<CR>

This will provide a way to start the Java debug server through coc.vim and then tell Vimspector which port to use to connect to the debug
server. It maps the `F1` key to kick things off, but you can change this to whatever you want.

#### Start the debug session

First, run a Java program with [remote debugging enabled](https://docs.oracle.com/javase/8/docs/technotes/guides/jpda/conninv.html#Invocation).
Be sure it is configured to pause and wait for a remote connection on port `5005` for this example work.

For a simple Java program. Create a `Hello.java` file with these contents.

    public class Hello {
      public static void main(String[] args) {
        System.out.println("Hello World!");
      }
    }

Next, run these commands from a shell to compile the program and then start it with remote debugging enabled.

    javac Hello.java
    java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=5005,suspend=y Hello


Or for another example [remote debugging Maven tests](https://maven.apache.org/surefire/maven-surefire-plugin/examples/debugging.html).

    mvn test -Dmaven.surefire.debug

If everything works correctly you will see this message.

> Listening for transport dt_socket at address: 5005

Now, open the file you want to debug in Vim and set a [breakpoint with Vimspector](https://github.com/puremourning/vimspector#mappings).

Finally, start the debug session in Vim by hitting the `F1` key or use your custom key combination if you have altered the
config from this example. This should result in Vimspector opening in a new tab with your Java program paused at the breakpoint you set.

That's it! You may now step debug your way through a Java program from within Vim.

*Note, if you use a Java debug port different than `5005` you will need to change that value in your `.vimspector.json` file. It is also
possible to configure this port dynamically in Vimspector in the same manner as the debug adapter port.*
