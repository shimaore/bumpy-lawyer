    L = require 'acoustic-line'

    macro = (name,content) ->
      if typeof name is 'object'
        L.macro name, content
      else
        L.macro {name}, content

    input = (pattern,break_on_match,content) ->
      if typeof break_on_match isnt 'boolean'
        content = break_on_match
        break_on_match = true

      L.input {pattern,break_on_match}, ->
        switch typeof content
          when 'function'
            L.match content
          when 'string'
            L.match ->
              play content
          else
            L.match ->
              for c in content
                play c

    play = (data) ->
      L.action function:'play', data:data

    phrase = (name,data) ->
      L.action function:'phrase', phrase:name, data:data

    module.exports = {macro,input,play,phrase}
