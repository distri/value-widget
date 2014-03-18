Value Widget
============

Tie a widget to an observable.

    Observable = require "observable"

    module.exports = (I={}) ->
      defaults I,
        width: 200
        height: 200
        value: null

      observable = Observable(I.value)
      widget = window.open I.url, "_blank", "width=#{I.width},height=#{I.height}"

      updating = false
      observable.observe (newValue) ->
        unless updating
          widget.postMessage
            method: "value"
            params: [newValue]
          , "*"

      listener = ({data, source}) ->
        if source is widget
          if value = data.value
            updating = true
            observable value
            updating = false

      window.addEventListener "message", listener

      widget.onunload = ->
        window.removeEventListener "message", listener

      window.addEventListener "unload", ->
        widget.close()

      return observable

Helpers
-------

    defaults = (target, objects...) ->
      for object in objects
        for name of object
          unless target.hasOwnProperty(name)
            target[name] = object[name]

      return target

Example
-------

    o = module.exports
      url: "http://distri.github.io/color-picker/"
      value: "hsl(180, 100%, 50%)"

    o.observe (v) ->
      console.log v
