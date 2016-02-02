    describe 'fr', ->
      it 'should build configuration', ->
        L = require 'acoustic-line'
        t = L.render require('../en').include {}
        assert 'string' is typeof t
        assert 0 < t.indexOf '<action function="play-file" data="voicemail/vm-received.wav"/>'
        assert 0 < t.indexOf '<input pattern="^0+$" break_on_match="true">'

    assert = require 'assert'
