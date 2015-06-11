Phrase generation for FreeSwitch
--------------------------------

A translation of the CCNQ `phrase` code into `acoustic-line`.
Used to generate FreeSwitch XML configuration for our `freeswitch-with-sounds` Docker.io container.

Usage
-----

Inside an `acoustic-line` configuration:

    section 'phrases', ->
      macros ->
        require('bumpy-lawyer/fr').include(sound_dir)
