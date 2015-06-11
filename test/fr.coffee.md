    describe 'fr', ->
      it 'should build configuration', ->
        L = require 'acoustic-line'
        t = L.render require('../fr').include('')
        assert 'string' is typeof t

    assert = require 'assert'
