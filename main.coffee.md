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

      if I.iframe 
        I.iframe.src = I.url if I.url
        widget = I.iframe.contentWindow
      else
        widget = window.open I.url, null, "width=#{I.width},height=#{I.height}"

      send = (data) ->
        widget.postMessage data, "*"

      update = (newValue) ->
        send
          method: "value"
          params: [newValue]

      observable.observe update

      listener = ({data, source}) ->
        return unless source is widget

        if data.status is "ready"
          if I.value?
            update(I.value)
        else if data.status is "unload"
          window.removeEventListener "message", listener
        else if value = data.value
          observable(value)

      window.addEventListener "message", listener

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

    applyStylesheet = (style, id="primary") ->
      styleNode = document.createElement("style")
      styleNode.innerHTML = style
      styleNode.id = id

      if previousStyleNode = document.head.querySelector("style##{id}")
        previousStyleNode.parentNode.removeChild(prevousStyleNode)

      document.head.appendChild(styleNode)

Example
-------

    if PACKAGE.name is "ROOT"
      applyStylesheet require "./demo"

      testFrame = document.createElement "iframe"

      document.body.appendChild testFrame

      o = module.exports
        iframe: testFrame
        url: "http://distri.github.io/text/"
        value: "hsl(180, 100%, 50%)"

      o.observe (v) ->
        console.log v
