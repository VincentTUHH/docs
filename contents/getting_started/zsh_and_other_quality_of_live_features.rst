.. _quality-of-life-getting-started:

Quality-of-Life Features
########################


Before we start, let's install some quality-of-life features to improve the user experience.

The Fast Way
************

.. code-block:: console

   $ sudo apt install -y zsh git curl wget byobu vim\
   && sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

Choose :code:`zsh` as default by hitting enter.

.. code-block:: console

   $ mkdir -p "$HOME/.zsh" && \
   git clone https://github.com/sindresorhus/pure.git "$HOME/.zsh/pure" && \
   echo 'fpath+=$HOME/.zsh/pure \nautoload -U promptinit; promptinit \nprompt pure' | cat - ~/.zshrc > temp && mv temp ~/.zshrc && \
   sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME=""/' ~/.zshrc && \
   git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions && \
   sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions)/' ~/.zshrc && \
   echo "zstyle ':prompt:pure:path' color 075\nzstyle ':prompt:pure:prompt:success' color 214\nzstyle ':prompt:pure:user' color 119\nzstyle ':prompt:pure:host' color 119\nZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=161'" >> ~/.zshrc && \
   echo "zstyle ':prompt:pure:path' color 075\nzstyle ':prompt:pure:prompt:success' color 214\nzstyle ':prompt:pure:user' color 119\nzstyle ':prompt:pure:host' color 119\nZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=161'" >> ~/.zshrc && \
   echo "export TERM=xterm-256color" >> ~/.zshrc && \
   source ~/.zshrc && \
   byobu-enable


Useful Shortcuts for Byobu
--------------------------

.. table::
   :widths: 5 15

   +--------------------------------------------------------+------------------------------------------------------------------+
   | Shortcut                                               | Function                                                         |
   +========================================================+==================================================================+
   |                        :kbd:`F2`                       | Open new terminal window                                         |
   +--------------------------------------------------------+------------------------------------------------------------------+
   |                        :kbd:`F3`                       | Move to previous window                                          |
   +--------------------------------------------------------+------------------------------------------------------------------+
   |                        :kbd:`F4`                       | Move to next window                                              |
   +--------------------------------------------------------+------------------------------------------------------------------+
   |                        :kbd:`F6`                       | Detach from this session                                         |
   +--------------------------------------------------------+------------------------------------------------------------------+
   | :kbd:`Ctrl` + :kbd:`A` and then :kbd:`Ctrl` + :kbd:`D` | Also Detach from this session.                                   |
   |                                                        |                                                                  |
   |                                                        | The first time you use this, you will be asked whether you       |
   |                                                        |                                                                  |
   |                                                        | want to use *screen* hotkeys or *emacs* hotkeys. Choose screen.  |
   +--------------------------------------------------------+------------------------------------------------------------------+
   |                 :kbd:`Alt` + :kbd:`F12`                | Turn mouse support for scrolling on/off                          |
   +--------------------------------------------------------+------------------------------------------------------------------+

Here's a more elaborate `cheat sheet <https://gist.github.com/inhumantsar/bf86ff1961cccdf8be06>`_.



The Good Way
************

.. note:: 

   This section is identical to "the fast way". We leave this in here for explanations.

zsh
---

We prefer using the Z shell (zsh). To install:

.. code-block:: console

   $ sudo apt install -y zsh git curl

Install `oh-my-zsh <https://ohmyz.sh/>`_:

.. code-block:: console

   $ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

Install the `Pure <https://github.com/sindresorhus/pure>`_ prompt:

.. code-block:: console

   # clone repo
   mkdir -p "$HOME/.zsh"
   git clone https://github.com/sindresorhus/pure.git "$HOME/.zsh/pure"

   # add path to ~/.zshrc, initialize prompt and choose pure (at the top of the file!)
   echo 'fpath+=$HOME/.zsh/pure \nautoload -U promptinit; promptinit \nprompt pure' | cat - ~/.zshrc > temp && mv temp ~/.zshrc

   # delete default theme entry
   sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME=""/' ~/.zshrc


Install `zsh-autosuggestions <https://github.com/zsh-users/zsh-autosuggestions>`_:

.. code-block:: console

   # clone repo
   git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

   # add plugin to list of plugins
   # assuming unchanged list containing only git, otherwise do this manually
   sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions)/' ~/.zshrc

Adjust colours of pure prompt and autosuggestions:

.. code-block:: console

   $ echo "zstyle ':prompt:pure:path' color 075\nzstyle ':prompt:pure:prompt:success' color 214\nzstyle ':prompt:pure:user' color 119\nzstyle ':prompt:pure:host' color 119\nZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=161'" >> ~/.zshrc

See the Pure `Zstyle options <https://github.com/sindresorhus/pure#zstyle-options>`_ and the `Xterm265 colour chart <https://upload.wikimedia.org/wikipedia/commons/1/15/Xterm_256color_chart.svg>`_ for other settings and colours.

To avoid not seeing the nice colors we just selected:

.. code-block:: console

   $ echo "export TERM=xterm-256color" >> ~/.zshrc


To apply changes:

.. code-block:: console

   $ source ~/.zshrc


Byobu
-----

To install `Byobu <https://www.byobu.org/>`_, a terminal multiplexer:

.. code-block:: console 

   $ sudo apt install byobu

Enable Byobu:

.. code-block:: console

   $ byobu-enable


VIM
---

To install vim:

.. code-block:: console

   $ sudo apt install vim




