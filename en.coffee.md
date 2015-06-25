    L = require 'acoustic-line'
    @include = (sound_dir) ->
      sound_path = "#{sound_dir}/en/us/callie"
      language = (name,content) ->
        L.language {name,'sound-path':sound_path}, content

      L.renderable ->
        language 'en', ->
          say()
          vm()
        language 'en-US', ->
          say()
          vm()

    {macro,input,play,phrase} = require './common'

Say
===

    say = ->

VoiceMail
=========

    vm = ->
