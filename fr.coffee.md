    L = require 'acoustic-line'
    @include = (sound_dir) ->
      sound_path = "#{sound_dir}/fr/fr/sibylle"
      language = (name,content) ->
        L.language {name,'sound-path':sound_path}, content

      L.renderable ->
        language 'fr', ->
          say()
          vm()
        language 'fr-FR', ->
          say()
          vm()

    {macro,input,play,phrase} = require './common'

Say
===

    say = ->
      macro 'say-single', ->

We didn't think of recording 'colon'.

        input '^:$', [
          'digits/2.wav'
          'digits/dot.wav'
        ]
        input '^([0-9])$', 'digits/$1.wav'
        input '^[*]$', 'digits/star.wav'
        input '^[#]$', 'digits/pound.wav'
        input '^[ ]$', 'phonetic-ascii/32.wav'
        input '^[.]$', 'phonetic-ascii/46.wav'
        input '^[@]$', 'phonetic-ascii/64.wav'
        for letter in [97..122]
          lowercase = String.fromCharCode letter
          uppercase =String.fromCharCode letter-32
          input "^[#{lowercase}#{uppercase}]$", "phonetic-ascii/#{letter}.wav"

      macro 'spell', ->
        input '^(.)$', ->
          phrase 'say-single', '$1'
        input '^(.)(.+)$', ->
          phrase 'say-single', '$1'
          phrase 'spell', '$2'

      macro 'say-iterated', ->
        input '^([0-9.#,*])$', ->
          phrase 'say-single', '$1'
        input '^([0-9.#,*])(.+)$', ->
          phrase 'say-single', '$1'
          phrase 'say-iterated', '$2'

      macro 'say-currency', ->

        input '^([0-9]+)$', ->
          phrase "say-number", "$1"
          play "currency/euro.wav"

        input "^([0-9]+)\.0([0-9])$", ->
          phrase "say-number", "$1"
          play "currency/euro.wav"
          phrase "say-number", "$2"
          play "currency/cent.wav"

        input "^([0-9]+)\.([1-9][0-9])$", ->
          phrase "say-number", "$1"
          play "currency/euro.wav"
          phrase "say-number", "$2"
          play "currency/cent.wav"

      macro "ip-addr", ->

IPv4 address

        input "^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$", ->
          phrase "say-number", "$1"
          play "digits/dot.wav"
          phrase "say-number", "$2"
          play "digits/dot.wav"
          phrase "say-number", "$3"
          play "digits/dot.wav"
          phrase "say-number", "$4"

IPv6 address

        input pattern="^([0-9a-f]{1,4})(\:.*)$", ->
          phrase "say-iterated", "$1"
          phrase "ip-addr", "$2"

        input pattern="^:([0-9a-f]{1,4})(.*)$", ->
          phrase "say", ":"
          phrase "say-iterated", "$1"
          phrase "ip-addr", "$2"

        input pattern="^::(.*)$", ->
          phrase "ip-addr", "::"
          phrase "ip-addr", "$1"

        input pattern="^::$", ->
          phrase "say", ":"
          phrase "say", ":"

      macro name:"say-counted",pause:"1", ->

workaround

        input "^([1-9])_(0)$", ->
          play "digits/h-$1$2.wav"

FIXME h-hundred is missing !!

        input "^(million|thousand|hundred)$", ->
          play "digits/h-$1.wav"

ignore leading zeros

        input "^0+(.*)$", ->
          phrase "say-counted", "$1"

9 digits counted

        input "^([1-9]|[1-9][0-9]|[1-9][0-9][0-9])000000f?n?$", ->
          phrase "say-number", "$1 counted million"

        input "^([1-9]|[1-9][0-9]|[1-9][0-9][0-9])([0-9]{6}f?n?)$", ->
          phrase "say-number", "$1 million counted $2n"

6 digits counted

        input "^1000f?n?$", ->
          phrase "say-number", "counted thousand"

        input "^([2-9]|[1-9][0-9]|[1-9][0-9][0-9])000f?n?$", ->
          phrase "say-number", "$1 counted thousand"

        input "^1([0-9]{3}f?n?)$", ->
          phrase "say-number", "thousand counted $1n"

        input "^([2-9]|[1-9][0-9]|[1-9][0-9][0-9])([0-9]{3}f?n?)$", ->
          phrase "say-number", "$1 thousand counted $2n"

3 digits counted

        input "^100f?n?$", ->
          phrase "say-number", "counted hundred"

        input "^([2-9])00f?n?$", ->
          phrase "say-number", "$1 counted hundred"

        input "^1([0-9][0-9]f?n?)$", ->
          phrase "say-number", "hundred counted $1n"

        input "^([2-9])([0-9][0-9]f?n?)$", ->
          phrase "say-number", "$1 hundred counted $2n"

two digits and less

        input "^1n$", ->
          play "digits/h-1n.wav" # FIXME Needs unième

        input "^1f$", ->
          play "digits/h-1f.wav" # première

        input "^([1-9]0|1[0-9]|[1-9])f?n?$", ->
          play "digits/h-$1.wav"

        input "^([23456])1f?n?$", ->
          phrase "say-number", "$1_0"
          play "digits/h-1n.wav" # "et unième"

        input "^([234568])([2-9])", ->
          phrase "say-number", "$1_0"
          play "digits/h-$2.wav"

        input "^71", ->
          play "digits/60.wav"
          play "currency/and.wav"
          play "digits/h-11.wav"

        input "^7([02-9])", ->
          play "digits/60.wav"
          play "digits/h-1$1.wav"

        input "^81", ->
          play "digits/80.wav"
          play "digits/h-1n.wav"

        input "^9([1-9])", ->
          play "digits/80.wav"
          play "digits/h-1$1.wav"

      macro name:"say-number", pause:"1", ->

        input "^ +(.*)$", ->
          phrase "say-number", "$1"

        input "^iterated *([^ ]+)(.*)$", ->
          phrase "say-iterated", "$1"
          phrase "say-number", "$2"

        input "^counted *([^ ]+)(.*)$", ->
          phrase "say-counted", "$1"
          phrase "say-number", "$2"

        input "^(million|thousand|hundred|point) *(.*)$", ->
          play "digits/$1.wav"
          phrase "say-number", "$2"

        input "^([^ ]+) +(.*)$", ->
          phrase "say-number", "$1"
          phrase "say-number", "$2"

        input "^-(.*)$", ->
          play "currency/minus.wav"
          phrase "say-number", "$1"

work around bug in switch_perform_substitution which prevents from
doing for example ${1}0, while $10 is interpreted as 'dollar-ten'.

        input "^([1-9])_(0)$", ->
          play "digits/$1$2.wav"

fractions
note: in french you would say "21 secondes virgule 1" so we do not
deal with feminine here

pronounce initial zeros
        input "^([0-9]*)\.([0]+)([1-9][0-9]*)$", ->
          phrase "say-number", "$1 point iterated $2 $3"

        input "^([0-9]*)\.([1-9][0-9]*)$", ->
          phrase "say-number", "$1 point $2"

ignore leading zeros

        input "^0f?$", ->
          play "digits/0.wav"
        input "^0+(.*)$", ->
          phrase "say-number", "$1"

9 digits or less

        input "^([1-9]|[1-9][0-9]|[1-9][0-9][0-9])000000f?$", ->
          phrase "say-number", "$1 million"

        input "^([1-9]|[1-9][0-9]|[1-9][0-9][0-9])([0-9]{6}f?)$", ->
          phrase "say-number", "$1 million $2"

six digits or less

        input "^1000f?$", ->
          phrase "say-number", "thousand"

        input "^([2-9]|[1-9][0-9]|[1-9][0-9][0-9])000f?$", ->
          phrase "say-number", "$1 thousand"

        input "^1([0-9]{3}f?)$", ->
          phrase "say-number", "thousand $1"

        input "^([2-9]|[1-9][0-9]|[1-9][0-9][0-9])([0-9]{3}f?)$", ->
          phrase "say-number", "$1 thousand $2"

three digits or less

        input "^100f?$", ->
          phrase "say-number", "hundred"

        input "^([2-9])00f?$", ->
          phrase "say-number", "$1 hundred"

        input "^1([0-9][0-9]f?)$", ->
          phrase "say-number", "hundred $1$2"

        input "^([2-9])([0-9][0-9]f?)$", ->
          phrase "say-number", "$1 hundred $2"

two digits or less

        input "^1f$", ->
          play "digits/1f.wav"

        input "^([1-9]0|1[0-9]|[1-9])f?$", ->
          play "digits/$1.wav"

        input "^([234568])([2-9])", ->
          phrase "say-number", "$1_0"
          play "digits/$2.wav"

        input "^([234568])1f", ->
          phrase "say-number", "$1_0"
          play "currency/and.wav"
          play "digits/1f.wav"

we recorded these specially.

        input "^(11|21|71|91)", ->
          play "digits/$1.wav"

        input "^([23456])1", ->
          phrase "say-number", "$1_0"
          play "generic/234.wav"

        input "^8(1.*)$", ->
          play "digits/80.wav"
          phrase "say-number", "$1"

        input "^7([02-9])", ->
        phrase "say-number", "60"
        phrase "say-number", "1$1"

        input "^9([0-9])", ->
          phrase "say-number", "80"
          phrase "say-number", "1$1"

      macro "say-day-of-month", ->

        input "^0?1$", ->
        phrase "say-counted", "1"

        input "^0?([2-9])$", ->
        phrase "say-number", "$1"

        input "^([123][0-9])$", ->
        phrase "say-number", "$1"

      macro "say-month", ->
        input "^0?1$", -> play "time/mon-0.wav"  
        input "^0?2$", -> play "time/mon-1.wav"  
        input "^0?3$", -> play "time/mon-2.wav"  
        input "^0?4$", -> play "time/mon-3.wav"  
        input "^0?5$", -> play "time/mon-4.wav"  
        input "^0?6$", -> play "time/mon-5.wav"  
        input "^0?7$", -> play "time/mon-6.wav"  
        input "^0?8$", -> play "time/mon-7.wav"  
        input "^0?9$", -> play "time/mon-8.wav"  
        input "^10$", -> play "time/mon-9.wav"  
        input "^11$", -> play "time/mon-10.wav"  
        input "^12$", -> play "time/mon-11.wav"  

      macro "say-time", ->

        input "^([0-9]{2}):00", ->
          phrase "say-number", "$1f"
          play "time/hour.wav"

        input "^([0-9]{2}):([0-9]{2})", ->
          phrase "say-number", "$1f"
          play "time/hour.wav"
          phrase "say-number", "$2f"
            # play "time/minute.wav"

      macro "say", ->

        input "^(.*) +(iterated|pronounced)$", ->
          phrase "say-iterated", "$1"

        input "^(.*) +counted$", ->
          phrase "say-counted", "$1"

        input "^(.*)( *€| +currency)$", ->
          phrase "say-currency", "$1"

ignore leading zeros

        input "^0(.*)$", ->
          phrase "say-number", "$1"

default

        input "^(.*) masculine?$", ->
          phrase "say", "$1"

        input "^(.)$", ->
          phrase "say-single", "$1"

VoiceMail
=========

    vm = ->

      macro "vm_say", ->
        input "^sorry$", ->
          play "misc/sorry.wav"
        input "^too short$", ->
          play "voicemail/vm-too-small.wav"
        input "^thank you$", ->
          play "ivr/ivr-thank_you_alt.wav"

      macro "voicemail_enter_id", ->
        input "(.*)", ->
          play "voicemail/vm-enter_id.wav"
          phrase "say", "$1 pronounced"

      macro "voicemail_enter_pass", ->
        input "(.*)", ->
          play "voicemail/vm-enter_pass.wav"
          phrase "say", "$1 pronounced"

      macro "voicemail_fail_auth", ->
        input "(.*)", ->
          play "voicemail/vm-fail_auth.wav"

      macro "voicemail_hello", ->
        input "(.*)", ->
          play "ivr/ivr-welcome.wav"

      macro "voicemail_goodbye", ->
        input "(.*)", ->
          play "voicemail/vm-goodbye.wav"

      macro "voicemail_abort", ->
        input "(.*)", ->
          play "voicemail/vm-abort.wav"

      macro "voicemail_message_count", ->
        input "^([^:]+):urgent-new", ->
          ###
          # TODO <action function="speak-text", "Vous avez $1 nouveaux messages urgents dans le répertoire ${voicemail_current_folder}."/> -->
          ###
        input "^0:new", ->
          play "voicemail/vm-you_have_neg.wav"
          play "generic/372.wav"
          play "voicemail/vm-new.wav"
          play "voicemail/vm-message.wav"
        input "^([^:]+):new", ->
          play "voicemail/vm-you_have.wav"
          phrase "say-number", "$1"
          play "voicemail/vm-new.wav"
          play "voicemail/vm-message.wav"
        input "^0:saved", ->
          play "voicemail/vm-you_have_neg.wav"
          play "generic/372.wav"
          play "voicemail/vm-message.wav"
          play "voicemail/vm-urgent.wav"
        input "^([^:]+):saved", ->
          play "voicemail/vm-you_have.wav"
          phrase "say-number", "$1"
          play "voicemail/vm-message.wav"
          play "voicemail/vm-urgent.wav"

      macro "voicemail_menu", ->
        input "^([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*])$", ->
          play "voicemail/vm-listen_new.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-listen_saved.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"
          play "voicemail/vm-advanced.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$3 pronounced"
          play "voicemail/vm-to_exit.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$4 pronounced"

      macro "voicemail_config_menu", ->
        input "^([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*])$", ->
          play "voicemail/vm-to_record_greeting.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          ###
          play "voicemail/vm-choose_greeting.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"
          ###
          play "voicemail/vm-record_name2.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$3 pronounced"
          play "voicemail/vm-change_password.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$4 pronounced"
          play "voicemail/vm-main_menu.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$5 pronounced"

      macro "voicemail_record_name", ->
        input "^(.*)$", ->
          play "voicemail/vm-record_name1.wav"

      macro "voicemail_record_file_check", ->
        input "^([0-9#*]):([0-9#*]):([0-9#*])$", ->
          play "voicemail/vm-listen_to_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-save_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"
          play "voicemail/vm-rerecord.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$3 pronounced"

      macro "voicemail_record_urgent_check", ->
        input "^([0-9#*]):([0-9#*])$", ->
          play "voicemail/vm-mark-urgent.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-continue.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"

      macro "voicemail_forward_prepend", ->
        input "^([0-9#*]):([0-9#*])$", ->
          play "voicemail/vm-forward_add_intro.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-send_message_now.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"

      macro "voicemail_forward_message_enter_extension", ->
        input "^([0-9#*])$", ->
          play "voicemail/vm-forward_enter_ext.wav"
          play "voicemail/vm-followed_by.wav"
          phrase "say", "$1 pronounced"

      macro "voicemail_invalid_extension", ->
        input "^(.*)$", ->
          play "voicemail/vm-that_was_an_invalid_ext.wav"

       macro "voicemail_listen_file_check", ->
        input "^([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*]):(.*)$", ->
          play "voicemail/vm-listen_to_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-save_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"
          play "voicemail/vm-delete_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$3 pronounced"
          play "voicemail/vm-forward_to_email.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$4 pronounced"
          play "voicemail/vm-return_call.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$5 pronounced"
          play "voicemail/vm-to_forward.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$6 pronounced"
        input "^([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*]):([0-9#*])$", ->
          play "voicemail/vm-listen_to_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-save_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$2 pronounced"
          play "voicemail/vm-delete_recording.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$3 pronounced"
          play "voicemail/vm-return_call.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$4 pronounced"
          play "voicemail/vm-to_forward.wav"
          play "voicemail/vm-press.wav"
          phrase "say", "$5 pronounced"

      macro "voicemail_choose_greeting", ->
        input "^(.*)$", ->
          play "voicemail/vm-choose_greeting_choose.wav"

      macro "voicemail_choose_greeting_fail", ->
        input "^(.*)$", ->
          play "voicemail/vm-choose_greeting_fail.wav"

      macro "voicemail_record_greeting", ->
        input "^(.*)$", ->
          play "voicemail/vm-record_greeting.wav"

      macro "voicemail_record_message", ->
        input "^(.*)$", ->
          play "voicemail/vm-record_message.wav"

      macro "voicemail_greeting_selected", ->
        input "^(\d+)$", ->
          play "voicemail/vm-greeting.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-selected.wav"

      macro "voicemail_play_greeting", ->
        input "^(.*)$", ->
          play "voicemail/vm-person.wav"
          phrase "say", "$1 pronounced"
          play "voicemail/vm-not_available.wav"

       macro "voicemail_unavailable", ->
        input "^(.*)$", ->
          play "voicemail/vm-not_available.wav"

      macro "voicemail_say_number", ->
        input "^(\d+)$", ->
          phrase "say", "$1 pronounced"

      macro "voicemail_say_message_number", ->
        input "^([a-z]+):(\d+)$", ->
          play "voicemail/vm-$1.wav" # new|urgent
          play "voicemail/vm-message_number.wav"
          phrase "say-number", "$2"

      macro "voicemail_say_phone_number", ->
        input "^(.*)$", ->
          phrase "say", "$1 pronounced"

      macro "voicemail_say_name", ->
        input "^(.*)$", ->
          phrase "say", "$1 pronounced"

      macro "voicemail_ack", ->
        input "^(too-small)$", ->
          play "voicemail/vm-too-small.wav"
        input "^(deleted|saved|emailed|marked-urgent)$", ->
          play "voicemail/vm-message.wav"
          play "voicemail/vm-$1.wav"

      macro "voicemail_say_date", ->
        input "^(.*)$", ->
          phrase "say-date", "$1"

      macro "voicemail_disk_quota_exceeded", ->
        input "^(.*)$", ->
          play "voicemail/vm-mailbox_full.wav"

      macro "valet_announce_ext", ->
        input "^([^\:]+):(.*)$", ->
          phrase "say", "$2 pronounced"

      macro "valet_lot_full", ->
        input "^(.*)$", ->
          play "tone_stream://%(275,10,600);%(275,100,300)"

      macro "valet_lot_empty", ->
        input "^(.*)$", ->
          play "tone_stream://%(275,10,600);%(275,100,300)"

      macro "message received", ->
        input "^([^:]+):([^:]*):[0-9]{4}-([0-9]{2})-([0-9]{2})T([0-9]{2}:[0-9]{2})", -> # message_number,caller_id,timestamp
          phrase "say-counted", "$1"
          play "voicemail/vm-message.wav"
          play "voicemail/vm-received.wav"
          phrase "say-day-of-month", "$4"
          phrase "say-month", "$3"
          play "time/at.wav"
          phrase "say-time", "$5"
