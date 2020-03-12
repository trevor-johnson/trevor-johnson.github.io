---
title: "NBA Game Simulation"
date: 2020-3-11
tags: [R, NBA, Simulation, Monte-Carlo, Sports]
excerpt: ""
---

<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">

<head>

<meta charset="utf-8" />
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<meta name="generator" content="pandoc" />
<meta http-equiv="X-UA-Compatible" content="IE=EDGE" />

<meta name="viewport" content="width=device-width, initial-scale=1">

<meta name="author" content="Trevor Johnson" />


<title>NBA Simulation</title>

<script>(function() {
  // If window.HTMLWidgets is already defined, then use it; otherwise create a
  // new object. This allows preceding code to set options that affect the
  // initialization process (though none currently exist).
  window.HTMLWidgets = window.HTMLWidgets || {};

  // See if we're running in a viewer pane. If not, we're in a web browser.
  var viewerMode = window.HTMLWidgets.viewerMode =
      /\bviewer_pane=1\b/.test(window.location);

  // See if we're running in Shiny mode. If not, it's a static document.
  // Note that static widgets can appear in both Shiny and static modes, but
  // obviously, Shiny widgets can only appear in Shiny apps/documents.
  var shinyMode = window.HTMLWidgets.shinyMode =
      typeof(window.Shiny) !== "undefined" && !!window.Shiny.outputBindings;

  // We can't count on jQuery being available, so we implement our own
  // version if necessary.
  function querySelectorAll(scope, selector) {
    if (typeof(jQuery) !== "undefined" && scope instanceof jQuery) {
      return scope.find(selector);
    }
    if (scope.querySelectorAll) {
      return scope.querySelectorAll(selector);
    }
  }

  function asArray(value) {
    if (value === null)
      return [];
    if ($.isArray(value))
      return value;
    return [value];
  }

  // Implement jQuery's extend
  function extend(target /*, ... */) {
    if (arguments.length == 1) {
      return target;
    }
    for (var i = 1; i < arguments.length; i++) {
      var source = arguments[i];
      for (var prop in source) {
        if (source.hasOwnProperty(prop)) {
          target[prop] = source[prop];
        }
      }
    }
    return target;
  }

  // IE8 doesn't support Array.forEach.
  function forEach(values, callback, thisArg) {
    if (values.forEach) {
      values.forEach(callback, thisArg);
    } else {
      for (var i = 0; i < values.length; i++) {
        callback.call(thisArg, values[i], i, values);
      }
    }
  }

  // Replaces the specified method with the return value of funcSource.
  //
  // Note that funcSource should not BE the new method, it should be a function
  // that RETURNS the new method. funcSource receives a single argument that is
  // the overridden method, it can be called from the new method. The overridden
  // method can be called like a regular function, it has the target permanently
  // bound to it so "this" will work correctly.
  function overrideMethod(target, methodName, funcSource) {
    var superFunc = target[methodName] || function() {};
    var superFuncBound = function() {
      return superFunc.apply(target, arguments);
    };
    target[methodName] = funcSource(superFuncBound);
  }

  // Add a method to delegator that, when invoked, calls
  // delegatee.methodName. If there is no such method on
  // the delegatee, but there was one on delegator before
  // delegateMethod was called, then the original version
  // is invoked instead.
  // For example:
  //
  // var a = {
  //   method1: function() { console.log('a1'); }
  //   method2: function() { console.log('a2'); }
  // };
  // var b = {
  //   method1: function() { console.log('b1'); }
  // };
  // delegateMethod(a, b, "method1");
  // delegateMethod(a, b, "method2");
  // a.method1();
  // a.method2();
  //
  // The output would be "b1", "a2".
  function delegateMethod(delegator, delegatee, methodName) {
    var inherited = delegator[methodName];
    delegator[methodName] = function() {
      var target = delegatee;
      var method = delegatee[methodName];

      // The method doesn't exist on the delegatee. Instead,
      // call the method on the delegator, if it exists.
      if (!method) {
        target = delegator;
        method = inherited;
      }

      if (method) {
        return method.apply(target, arguments);
      }
    };
  }

  // Implement a vague facsimilie of jQuery's data method
  function elementData(el, name, value) {
    if (arguments.length == 2) {
      return el["htmlwidget_data_" + name];
    } else if (arguments.length == 3) {
      el["htmlwidget_data_" + name] = value;
      return el;
    } else {
      throw new Error("Wrong number of arguments for elementData: " +
        arguments.length);
    }
  }

  // http://stackoverflow.com/questions/3446170/escape-string-for-use-in-javascript-regex
  function escapeRegExp(str) {
    return str.replace(/[\-\[\]\/\{\}\(\)\*\+\?\.\\\^\$\|]/g, "\\$&");
  }

  function hasClass(el, className) {
    var re = new RegExp("\\b" + escapeRegExp(className) + "\\b");
    return re.test(el.className);
  }

  // elements - array (or array-like object) of HTML elements
  // className - class name to test for
  // include - if true, only return elements with given className;
  //   if false, only return elements *without* given className
  function filterByClass(elements, className, include) {
    var results = [];
    for (var i = 0; i < elements.length; i++) {
      if (hasClass(elements[i], className) == include)
        results.push(elements[i]);
    }
    return results;
  }

  function on(obj, eventName, func) {
    if (obj.addEventListener) {
      obj.addEventListener(eventName, func, false);
    } else if (obj.attachEvent) {
      obj.attachEvent(eventName, func);
    }
  }

  function off(obj, eventName, func) {
    if (obj.removeEventListener)
      obj.removeEventListener(eventName, func, false);
    else if (obj.detachEvent) {
      obj.detachEvent(eventName, func);
    }
  }

  // Translate array of values to top/right/bottom/left, as usual with
  // the "padding" CSS property
  // https://developer.mozilla.org/en-US/docs/Web/CSS/padding
  function unpackPadding(value) {
    if (typeof(value) === "number")
      value = [value];
    if (value.length === 1) {
      return {top: value[0], right: value[0], bottom: value[0], left: value[0]};
    }
    if (value.length === 2) {
      return {top: value[0], right: value[1], bottom: value[0], left: value[1]};
    }
    if (value.length === 3) {
      return {top: value[0], right: value[1], bottom: value[2], left: value[1]};
    }
    if (value.length === 4) {
      return {top: value[0], right: value[1], bottom: value[2], left: value[3]};
    }
  }

  // Convert an unpacked padding object to a CSS value
  function paddingToCss(paddingObj) {
    return paddingObj.top + "px " + paddingObj.right + "px " + paddingObj.bottom + "px " + paddingObj.left + "px";
  }

  // Makes a number suitable for CSS
  function px(x) {
    if (typeof(x) === "number")
      return x + "px";
    else
      return x;
  }

  // Retrieves runtime widget sizing information for an element.
  // The return value is either null, or an object with fill, padding,
  // defaultWidth, defaultHeight fields.
  function sizingPolicy(el) {
    var sizingEl = document.querySelector("script[data-for='" + el.id + "'][type='application/htmlwidget-sizing']");
    if (!sizingEl)
      return null;
    var sp = JSON.parse(sizingEl.textContent || sizingEl.text || "{}");
    if (viewerMode) {
      return sp.viewer;
    } else {
      return sp.browser;
    }
  }

  // @param tasks Array of strings (or falsy value, in which case no-op).
  //   Each element must be a valid JavaScript expression that yields a
  //   function. Or, can be an array of objects with "code" and "data"
  //   properties; in this case, the "code" property should be a string
  //   of JS that's an expr that yields a function, and "data" should be
  //   an object that will be added as an additional argument when that
  //   function is called.
  // @param target The object that will be "this" for each function
  //   execution.
  // @param args Array of arguments to be passed to the functions. (The
  //   same arguments will be passed to all functions.)
  function evalAndRun(tasks, target, args) {
    if (tasks) {
      forEach(tasks, function(task) {
        var theseArgs = args;
        if (typeof(task) === "object") {
          theseArgs = theseArgs.concat([task.data]);
          task = task.code;
        }
        var taskFunc = eval("(" + task + ")");
        if (typeof(taskFunc) !== "function") {
          throw new Error("Task must be a function! Source:\n" + task);
        }
        taskFunc.apply(target, theseArgs);
      });
    }
  }

  function initSizing(el) {
    var sizing = sizingPolicy(el);
    if (!sizing)
      return;

    var cel = document.getElementById("htmlwidget_container");
    if (!cel)
      return;

    if (typeof(sizing.padding) !== "undefined") {
      document.body.style.margin = "0";
      document.body.style.padding = paddingToCss(unpackPadding(sizing.padding));
    }

    if (sizing.fill) {
      document.body.style.overflow = "hidden";
      document.body.style.width = "100%";
      document.body.style.height = "100%";
      document.documentElement.style.width = "100%";
      document.documentElement.style.height = "100%";
      if (cel) {
        cel.style.position = "absolute";
        var pad = unpackPadding(sizing.padding);
        cel.style.top = pad.top + "px";
        cel.style.right = pad.right + "px";
        cel.style.bottom = pad.bottom + "px";
        cel.style.left = pad.left + "px";
        el.style.width = "100%";
        el.style.height = "100%";
      }

      return {
        getWidth: function() { return cel.offsetWidth; },
        getHeight: function() { return cel.offsetHeight; }
      };

    } else {
      el.style.width = px(sizing.width);
      el.style.height = px(sizing.height);

      return {
        getWidth: function() { return el.offsetWidth; },
        getHeight: function() { return el.offsetHeight; }
      };
    }
  }

  // Default implementations for methods
  var defaults = {
    find: function(scope) {
      return querySelectorAll(scope, "." + this.name);
    },
    renderError: function(el, err) {
      var $el = $(el);

      this.clearError(el);

      // Add all these error classes, as Shiny does
      var errClass = "shiny-output-error";
      if (err.type !== null) {
        // use the classes of the error condition as CSS class names
        errClass = errClass + " " + $.map(asArray(err.type), function(type) {
          return errClass + "-" + type;
        }).join(" ");
      }
      errClass = errClass + " htmlwidgets-error";

      // Is el inline or block? If inline or inline-block, just display:none it
      // and add an inline error.
      var display = $el.css("display");
      $el.data("restore-display-mode", display);

      if (display === "inline" || display === "inline-block") {
        $el.hide();
        if (err.message !== "") {
          var errorSpan = $("<span>").addClass(errClass);
          errorSpan.text(err.message);
          $el.after(errorSpan);
        }
      } else if (display === "block") {
        // If block, add an error just after the el, set visibility:none on the
        // el, and position the error to be on top of the el.
        // Mark it with a unique ID and CSS class so we can remove it later.
        $el.css("visibility", "hidden");
        if (err.message !== "") {
          var errorDiv = $("<div>").addClass(errClass).css("position", "absolute")
            .css("top", el.offsetTop)
            .css("left", el.offsetLeft)
            // setting width can push out the page size, forcing otherwise
            // unnecessary scrollbars to appear and making it impossible for
            // the element to shrink; so use max-width instead
            .css("maxWidth", el.offsetWidth)
            .css("height", el.offsetHeight);
          errorDiv.text(err.message);
          $el.after(errorDiv);

          // Really dumb way to keep the size/position of the error in sync with
          // the parent element as the window is resized or whatever.
          var intId = setInterval(function() {
            if (!errorDiv[0].parentElement) {
              clearInterval(intId);
              return;
            }
            errorDiv
              .css("top", el.offsetTop)
              .css("left", el.offsetLeft)
              .css("maxWidth", el.offsetWidth)
              .css("height", el.offsetHeight);
          }, 500);
        }
      }
    },
    clearError: function(el) {
      var $el = $(el);
      var display = $el.data("restore-display-mode");
      $el.data("restore-display-mode", null);

      if (display === "inline" || display === "inline-block") {
        if (display)
          $el.css("display", display);
        $(el.nextSibling).filter(".htmlwidgets-error").remove();
      } else if (display === "block"){
        $el.css("visibility", "inherit");
        $(el.nextSibling).filter(".htmlwidgets-error").remove();
      }
    },
    sizing: {}
  };

  // Called by widget bindings to register a new type of widget. The definition
  // object can contain the following properties:
  // - name (required) - A string indicating the binding name, which will be
  //   used by default as the CSS classname to look for.
  // - initialize (optional) - A function(el) that will be called once per
  //   widget element; if a value is returned, it will be passed as the third
  //   value to renderValue.
  // - renderValue (required) - A function(el, data, initValue) that will be
  //   called with data. Static contexts will cause this to be called once per
  //   element; Shiny apps will cause this to be called multiple times per
  //   element, as the data changes.
  window.HTMLWidgets.widget = function(definition) {
    if (!definition.name) {
      throw new Error("Widget must have a name");
    }
    if (!definition.type) {
      throw new Error("Widget must have a type");
    }
    // Currently we only support output widgets
    if (definition.type !== "output") {
      throw new Error("Unrecognized widget type '" + definition.type + "'");
    }
    // TODO: Verify that .name is a valid CSS classname

    // Support new-style instance-bound definitions. Old-style class-bound
    // definitions have one widget "object" per widget per type/class of
    // widget; the renderValue and resize methods on such widget objects
    // take el and instance arguments, because the widget object can't
    // store them. New-style instance-bound definitions have one widget
    // object per widget instance; the definition that's passed in doesn't
    // provide renderValue or resize methods at all, just the single method
    //   factory(el, width, height)
    // which returns an object that has renderValue(x) and resize(w, h).
    // This enables a far more natural programming style for the widget
    // author, who can store per-instance state using either OO-style
    // instance fields or functional-style closure variables (I guess this
    // is in contrast to what can only be called C-style pseudo-OO which is
    // what we required before).
    if (definition.factory) {
      definition = createLegacyDefinitionAdapter(definition);
    }

    if (!definition.renderValue) {
      throw new Error("Widget must have a renderValue function");
    }

    // For static rendering (non-Shiny), use a simple widget registration
    // scheme. We also use this scheme for Shiny apps/documents that also
    // contain static widgets.
    window.HTMLWidgets.widgets = window.HTMLWidgets.widgets || [];
    // Merge defaults into the definition; don't mutate the original definition.
    var staticBinding = extend({}, defaults, definition);
    overrideMethod(staticBinding, "find", function(superfunc) {
      return function(scope) {
        var results = superfunc(scope);
        // Filter out Shiny outputs, we only want the static kind
        return filterByClass(results, "html-widget-output", false);
      };
    });
    window.HTMLWidgets.widgets.push(staticBinding);

    if (shinyMode) {
      // Shiny is running. Register the definition with an output binding.
      // The definition itself will not be the output binding, instead
      // we will make an output binding object that delegates to the
      // definition. This is because we foolishly used the same method
      // name (renderValue) for htmlwidgets definition and Shiny bindings
      // but they actually have quite different semantics (the Shiny
      // bindings receive data that includes lots of metadata that it
      // strips off before calling htmlwidgets renderValue). We can't
      // just ignore the difference because in some widgets it's helpful
      // to call this.renderValue() from inside of resize(), and if
      // we're not delegating, then that call will go to the Shiny
      // version instead of the htmlwidgets version.

      // Merge defaults with definition, without mutating either.
      var bindingDef = extend({}, defaults, definition);

      // This object will be our actual Shiny binding.
      var shinyBinding = new Shiny.OutputBinding();

      // With a few exceptions, we'll want to simply use the bindingDef's
      // version of methods if they are available, otherwise fall back to
      // Shiny's defaults. NOTE: If Shiny's output bindings gain additional
      // methods in the future, and we want them to be overrideable by
      // HTMLWidget binding definitions, then we'll need to add them to this
      // list.
      delegateMethod(shinyBinding, bindingDef, "getId");
      delegateMethod(shinyBinding, bindingDef, "onValueChange");
      delegateMethod(shinyBinding, bindingDef, "onValueError");
      delegateMethod(shinyBinding, bindingDef, "renderError");
      delegateMethod(shinyBinding, bindingDef, "clearError");
      delegateMethod(shinyBinding, bindingDef, "showProgress");

      // The find, renderValue, and resize are handled differently, because we
      // want to actually decorate the behavior of the bindingDef methods.

      shinyBinding.find = function(scope) {
        var results = bindingDef.find(scope);

        // Only return elements that are Shiny outputs, not static ones
        var dynamicResults = results.filter(".html-widget-output");

        // It's possible that whatever caused Shiny to think there might be
        // new dynamic outputs, also caused there to be new static outputs.
        // Since there might be lots of different htmlwidgets bindings, we
        // schedule execution for later--no need to staticRender multiple
        // times.
        if (results.length !== dynamicResults.length)
          scheduleStaticRender();

        return dynamicResults;
      };

      // Wrap renderValue to handle initialization, which unfortunately isn't
      // supported natively by Shiny at the time of this writing.

      shinyBinding.renderValue = function(el, data) {
        Shiny.renderDependencies(data.deps);
        // Resolve strings marked as javascript literals to objects
        if (!(data.evals instanceof Array)) data.evals = [data.evals];
        for (var i = 0; data.evals && i < data.evals.length; i++) {
          window.HTMLWidgets.evaluateStringMember(data.x, data.evals[i]);
        }
        if (!bindingDef.renderOnNullValue) {
          if (data.x === null) {
            el.style.visibility = "hidden";
            return;
          } else {
            el.style.visibility = "inherit";
          }
        }
        if (!elementData(el, "initialized")) {
          initSizing(el);

          elementData(el, "initialized", true);
          if (bindingDef.initialize) {
            var result = bindingDef.initialize(el, el.offsetWidth,
              el.offsetHeight);
            elementData(el, "init_result", result);
          }
        }
        bindingDef.renderValue(el, data.x, elementData(el, "init_result"));
        evalAndRun(data.jsHooks.render, elementData(el, "init_result"), [el, data.x]);
      };

      // Only override resize if bindingDef implements it
      if (bindingDef.resize) {
        shinyBinding.resize = function(el, width, height) {
          // Shiny can call resize before initialize/renderValue have been
          // called, which doesn't make sense for widgets.
          if (elementData(el, "initialized")) {
            bindingDef.resize(el, width, height, elementData(el, "init_result"));
          }
        };
      }

      Shiny.outputBindings.register(shinyBinding, bindingDef.name);
    }
  };

  var scheduleStaticRenderTimerId = null;
  function scheduleStaticRender() {
    if (!scheduleStaticRenderTimerId) {
      scheduleStaticRenderTimerId = setTimeout(function() {
        scheduleStaticRenderTimerId = null;
        window.HTMLWidgets.staticRender();
      }, 1);
    }
  }

  // Render static widgets after the document finishes loading
  // Statically render all elements that are of this widget's class
  window.HTMLWidgets.staticRender = function() {
    var bindings = window.HTMLWidgets.widgets || [];
    forEach(bindings, function(binding) {
      var matches = binding.find(document.documentElement);
      forEach(matches, function(el) {
        var sizeObj = initSizing(el, binding);

        if (hasClass(el, "html-widget-static-bound"))
          return;
        el.className = el.className + " html-widget-static-bound";

        var initResult;
        if (binding.initialize) {
          initResult = binding.initialize(el,
            sizeObj ? sizeObj.getWidth() : el.offsetWidth,
            sizeObj ? sizeObj.getHeight() : el.offsetHeight
          );
          elementData(el, "init_result", initResult);
        }

        if (binding.resize) {
          var lastSize = {
            w: sizeObj ? sizeObj.getWidth() : el.offsetWidth,
            h: sizeObj ? sizeObj.getHeight() : el.offsetHeight
          };
          var resizeHandler = function(e) {
            var size = {
              w: sizeObj ? sizeObj.getWidth() : el.offsetWidth,
              h: sizeObj ? sizeObj.getHeight() : el.offsetHeight
            };
            if (size.w === 0 && size.h === 0)
              return;
            if (size.w === lastSize.w && size.h === lastSize.h)
              return;
            lastSize = size;
            binding.resize(el, size.w, size.h, initResult);
          };

          on(window, "resize", resizeHandler);

          // This is needed for cases where we're running in a Shiny
          // app, but the widget itself is not a Shiny output, but
          // rather a simple static widget. One example of this is
          // an rmarkdown document that has runtime:shiny and widget
          // that isn't in a render function. Shiny only knows to
          // call resize handlers for Shiny outputs, not for static
          // widgets, so we do it ourselves.
          if (window.jQuery) {
            window.jQuery(document).on(
              "shown.htmlwidgets shown.bs.tab.htmlwidgets shown.bs.collapse.htmlwidgets",
              resizeHandler
            );
            window.jQuery(document).on(
              "hidden.htmlwidgets hidden.bs.tab.htmlwidgets hidden.bs.collapse.htmlwidgets",
              resizeHandler
            );
          }

          // This is needed for the specific case of ioslides, which
          // flips slides between display:none and display:block.
          // Ideally we would not have to have ioslide-specific code
          // here, but rather have ioslides raise a generic event,
          // but the rmarkdown package just went to CRAN so the
          // window to getting that fixed may be long.
          if (window.addEventListener) {
            // It's OK to limit this to window.addEventListener
            // browsers because ioslides itself only supports
            // such browsers.
            on(document, "slideenter", resizeHandler);
            on(document, "slideleave", resizeHandler);
          }
        }

        var scriptData = document.querySelector("script[data-for='" + el.id + "'][type='application/json']");
        if (scriptData) {
          var data = JSON.parse(scriptData.textContent || scriptData.text);
          // Resolve strings marked as javascript literals to objects
          if (!(data.evals instanceof Array)) data.evals = [data.evals];
          for (var k = 0; data.evals && k < data.evals.length; k++) {
            window.HTMLWidgets.evaluateStringMember(data.x, data.evals[k]);
          }
          binding.renderValue(el, data.x, initResult);
          evalAndRun(data.jsHooks.render, initResult, [el, data.x]);
        }
      });
    });

    invokePostRenderHandlers();
  }

  // Wait until after the document has loaded to render the widgets.
  if (document.addEventListener) {
    document.addEventListener("DOMContentLoaded", function() {
      document.removeEventListener("DOMContentLoaded", arguments.callee, false);
      window.HTMLWidgets.staticRender();
    }, false);
  } else if (document.attachEvent) {
    document.attachEvent("onreadystatechange", function() {
      if (document.readyState === "complete") {
        document.detachEvent("onreadystatechange", arguments.callee);
        window.HTMLWidgets.staticRender();
      }
    });
  }


  window.HTMLWidgets.getAttachmentUrl = function(depname, key) {
    // If no key, default to the first item
    if (typeof(key) === "undefined")
      key = 1;

    var link = document.getElementById(depname + "-" + key + "-attachment");
    if (!link) {
      throw new Error("Attachment " + depname + "/" + key + " not found in document");
    }
    return link.getAttribute("href");
  };

  window.HTMLWidgets.dataframeToD3 = function(df) {
    var names = [];
    var length;
    for (var name in df) {
        if (df.hasOwnProperty(name))
            names.push(name);
        if (typeof(df[name]) !== "object" || typeof(df[name].length) === "undefined") {
            throw new Error("All fields must be arrays");
        } else if (typeof(length) !== "undefined" && length !== df[name].length) {
            throw new Error("All fields must be arrays of the same length");
        }
        length = df[name].length;
    }
    var results = [];
    var item;
    for (var row = 0; row < length; row++) {
        item = {};
        for (var col = 0; col < names.length; col++) {
            item[names[col]] = df[names[col]][row];
        }
        results.push(item);
    }
    return results;
  };

  window.HTMLWidgets.transposeArray2D = function(array) {
      if (array.length === 0) return array;
      var newArray = array[0].map(function(col, i) {
          return array.map(function(row) {
              return row[i]
          })
      });
      return newArray;
  };
  // Split value at splitChar, but allow splitChar to be escaped
  // using escapeChar. Any other characters escaped by escapeChar
  // will be included as usual (including escapeChar itself).
  function splitWithEscape(value, splitChar, escapeChar) {
    var results = [];
    var escapeMode = false;
    var currentResult = "";
    for (var pos = 0; pos < value.length; pos++) {
      if (!escapeMode) {
        if (value[pos] === splitChar) {
          results.push(currentResult);
          currentResult = "";
        } else if (value[pos] === escapeChar) {
          escapeMode = true;
        } else {
          currentResult += value[pos];
        }
      } else {
        currentResult += value[pos];
        escapeMode = false;
      }
    }
    if (currentResult !== "") {
      results.push(currentResult);
    }
    return results;
  }
  // Function authored by Yihui/JJ Allaire
  window.HTMLWidgets.evaluateStringMember = function(o, member) {
    var parts = splitWithEscape(member, '.', '\\');
    for (var i = 0, l = parts.length; i < l; i++) {
      var part = parts[i];
      // part may be a character or 'numeric' member name
      if (o !== null && typeof o === "object" && part in o) {
        if (i == (l - 1)) { // if we are at the end of the line then evalulate
          if (typeof o[part] === "string")
            o[part] = eval("(" + o[part] + ")");
        } else { // otherwise continue to next embedded object
          o = o[part];
        }
      }
    }
  };

  // Retrieve the HTMLWidget instance (i.e. the return value of an
  // HTMLWidget binding's initialize() or factory() function)
  // associated with an element, or null if none.
  window.HTMLWidgets.getInstance = function(el) {
    return elementData(el, "init_result");
  };

  // Finds the first element in the scope that matches the selector,
  // and returns the HTMLWidget instance (i.e. the return value of
  // an HTMLWidget binding's initialize() or factory() function)
  // associated with that element, if any. If no element matches the
  // selector, or the first matching element has no HTMLWidget
  // instance associated with it, then null is returned.
  //
  // The scope argument is optional, and defaults to window.document.
  window.HTMLWidgets.find = function(scope, selector) {
    if (arguments.length == 1) {
      selector = scope;
      scope = document;
    }

    var el = scope.querySelector(selector);
    if (el === null) {
      return null;
    } else {
      return window.HTMLWidgets.getInstance(el);
    }
  };

  // Finds all elements in the scope that match the selector, and
  // returns the HTMLWidget instances (i.e. the return values of
  // an HTMLWidget binding's initialize() or factory() function)
  // associated with the elements, in an array. If elements that
  // match the selector don't have an associated HTMLWidget
  // instance, the returned array will contain nulls.
  //
  // The scope argument is optional, and defaults to window.document.
  window.HTMLWidgets.findAll = function(scope, selector) {
    if (arguments.length == 1) {
      selector = scope;
      scope = document;
    }

    var nodes = scope.querySelectorAll(selector);
    var results = [];
    for (var i = 0; i < nodes.length; i++) {
      results.push(window.HTMLWidgets.getInstance(nodes[i]));
    }
    return results;
  };

  var postRenderHandlers = [];
  function invokePostRenderHandlers() {
    while (postRenderHandlers.length) {
      var handler = postRenderHandlers.shift();
      if (handler) {
        handler();
      }
    }
  }

  // Register the given callback function to be invoked after the
  // next time static widgets are rendered.
  window.HTMLWidgets.addPostRenderHandler = function(callback) {
    postRenderHandlers.push(callback);
  };

  // Takes a new-style instance-bound definition, and returns an
  // old-style class-bound definition. This saves us from having
  // to rewrite all the logic in this file to accomodate both
  // types of definitions.
  function createLegacyDefinitionAdapter(defn) {
    var result = {
      name: defn.name,
      type: defn.type,
      initialize: function(el, width, height) {
        return defn.factory(el, width, height);
      },
      renderValue: function(el, x, instance) {
        return instance.renderValue(x);
      },
      resize: function(el, width, height, instance) {
        return instance.resize(width, height);
      }
    };

    if (defn.find)
      result.find = defn.find;
    if (defn.renderError)
      result.renderError = defn.renderError;
    if (defn.clearError)
      result.clearError = defn.clearError;

    return result;
  }
})();

</script>
<script>/*! jQuery v1.12.4 | (c) jQuery Foundation | jquery.org/license */
!function(a,b){"object"==typeof module&&"object"==typeof module.exports?module.exports=a.document?b(a,!0):function(a){if(!a.document)throw new Error("jQuery requires a window with a document");return b(a)}:b(a)}("undefined"!=typeof window?window:this,function(a,b){var c=[],d=a.document,e=c.slice,f=c.concat,g=c.push,h=c.indexOf,i={},j=i.toString,k=i.hasOwnProperty,l={},m="1.12.4",n=function(a,b){return new n.fn.init(a,b)},o=/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g,p=/^-ms-/,q=/-([\da-z])/gi,r=function(a,b){return b.toUpperCase()};n.fn=n.prototype={jquery:m,constructor:n,selector:"",length:0,toArray:function(){return e.call(this)},get:function(a){return null!=a?0>a?this[a+this.length]:this[a]:e.call(this)},pushStack:function(a){var b=n.merge(this.constructor(),a);return b.prevObject=this,b.context=this.context,b},each:function(a){return n.each(this,a)},map:function(a){return this.pushStack(n.map(this,function(b,c){return a.call(b,c,b)}))},slice:function(){return this.pushStack(e.apply(this,arguments))},first:function(){return this.eq(0)},last:function(){return this.eq(-1)},eq:function(a){var b=this.length,c=+a+(0>a?b:0);return this.pushStack(c>=0&&b>c?[this[c]]:[])},end:function(){return this.prevObject||this.constructor()},push:g,sort:c.sort,splice:c.splice},n.extend=n.fn.extend=function(){var a,b,c,d,e,f,g=arguments[0]||{},h=1,i=arguments.length,j=!1;for("boolean"==typeof g&&(j=g,g=arguments[h]||{},h++),"object"==typeof g||n.isFunction(g)||(g={}),h===i&&(g=this,h--);i>h;h++)if(null!=(e=arguments[h]))for(d in e)a=g[d],c=e[d],g!==c&&(j&&c&&(n.isPlainObject(c)||(b=n.isArray(c)))?(b?(b=!1,f=a&&n.isArray(a)?a:[]):f=a&&n.isPlainObject(a)?a:{},g[d]=n.extend(j,f,c)):void 0!==c&&(g[d]=c));return g},n.extend({expando:"jQuery"+(m+Math.random()).replace(/\D/g,""),isReady:!0,error:function(a){throw new Error(a)},noop:function(){},isFunction:function(a){return"function"===n.type(a)},isArray:Array.isArray||function(a){return"array"===n.type(a)},isWindow:function(a){return null!=a&&a==a.window},isNumeric:function(a){var b=a&&a.toString();return!n.isArray(a)&&b-parseFloat(b)+1>=0},isEmptyObject:function(a){var b;for(b in a)return!1;return!0},isPlainObject:function(a){var b;if(!a||"object"!==n.type(a)||a.nodeType||n.isWindow(a))return!1;try{if(a.constructor&&!k.call(a,"constructor")&&!k.call(a.constructor.prototype,"isPrototypeOf"))return!1}catch(c){return!1}if(!l.ownFirst)for(b in a)return k.call(a,b);for(b in a);return void 0===b||k.call(a,b)},type:function(a){return null==a?a+"":"object"==typeof a||"function"==typeof a?i[j.call(a)]||"object":typeof a},globalEval:function(b){b&&n.trim(b)&&(a.execScript||function(b){a.eval.call(a,b)})(b)},camelCase:function(a){return a.replace(p,"ms-").replace(q,r)},nodeName:function(a,b){return a.nodeName&&a.nodeName.toLowerCase()===b.toLowerCase()},each:function(a,b){var c,d=0;if(s(a)){for(c=a.length;c>d;d++)if(b.call(a[d],d,a[d])===!1)break}else for(d in a)if(b.call(a[d],d,a[d])===!1)break;return a},trim:function(a){return null==a?"":(a+"").replace(o,"")},makeArray:function(a,b){var c=b||[];return null!=a&&(s(Object(a))?n.merge(c,"string"==typeof a?[a]:a):g.call(c,a)),c},inArray:function(a,b,c){var d;if(b){if(h)return h.call(b,a,c);for(d=b.length,c=c?0>c?Math.max(0,d+c):c:0;d>c;c++)if(c in b&&b[c]===a)return c}return-1},merge:function(a,b){var c=+b.length,d=0,e=a.length;while(c>d)a[e++]=b[d++];if(c!==c)while(void 0!==b[d])a[e++]=b[d++];return a.length=e,a},grep:function(a,b,c){for(var d,e=[],f=0,g=a.length,h=!c;g>f;f++)d=!b(a[f],f),d!==h&&e.push(a[f]);return e},map:function(a,b,c){var d,e,g=0,h=[];if(s(a))for(d=a.length;d>g;g++)e=b(a[g],g,c),null!=e&&h.push(e);else for(g in a)e=b(a[g],g,c),null!=e&&h.push(e);return f.apply([],h)},guid:1,proxy:function(a,b){var c,d,f;return"string"==typeof b&&(f=a[b],b=a,a=f),n.isFunction(a)?(c=e.call(arguments,2),d=function(){return a.apply(b||this,c.concat(e.call(arguments)))},d.guid=a.guid=a.guid||n.guid++,d):void 0},now:function(){return+new Date},support:l}),"function"==typeof Symbol&&(n.fn[Symbol.iterator]=c[Symbol.iterator]),n.each("Boolean Number String Function Array Date RegExp Object Error Symbol".split(" "),function(a,b){i["[object "+b+"]"]=b.toLowerCase()});function s(a){var b=!!a&&"length"in a&&a.length,c=n.type(a);return"function"===c||n.isWindow(a)?!1:"array"===c||0===b||"number"==typeof b&&b>0&&b-1 in a}var t=function(a){var b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u="sizzle"+1*new Date,v=a.document,w=0,x=0,y=ga(),z=ga(),A=ga(),B=function(a,b){return a===b&&(l=!0),0},C=1<<31,D={}.hasOwnProperty,E=[],F=E.pop,G=E.push,H=E.push,I=E.slice,J=function(a,b){for(var c=0,d=a.length;d>c;c++)if(a[c]===b)return c;return-1},K="checked|selected|async|autofocus|autoplay|controls|defer|disabled|hidden|ismap|loop|multiple|open|readonly|required|scoped",L="[\\x20\\t\\r\\n\\f]",M="(?:\\\\.|[\\w-]|[^\\x00-\\xa0])+",N="\\["+L+"*("+M+")(?:"+L+"*([*^$|!~]?=)"+L+"*(?:'((?:\\\\.|[^\\\\'])*)'|\"((?:\\\\.|[^\\\\\"])*)\"|("+M+"))|)"+L+"*\\]",O=":("+M+")(?:\\((('((?:\\\\.|[^\\\\'])*)'|\"((?:\\\\.|[^\\\\\"])*)\")|((?:\\\\.|[^\\\\()[\\]]|"+N+")*)|.*)\\)|)",P=new RegExp(L+"+","g"),Q=new RegExp("^"+L+"+|((?:^|[^\\\\])(?:\\\\.)*)"+L+"+$","g"),R=new RegExp("^"+L+"*,"+L+"*"),S=new RegExp("^"+L+"*([>+~]|"+L+")"+L+"*"),T=new RegExp("="+L+"*([^\\]'\"]*?)"+L+"*\\]","g"),U=new RegExp(O),V=new RegExp("^"+M+"$"),W={ID:new RegExp("^#("+M+")"),CLASS:new RegExp("^\\.("+M+")"),TAG:new RegExp("^("+M+"|[*])"),ATTR:new RegExp("^"+N),PSEUDO:new RegExp("^"+O),CHILD:new RegExp("^:(only|first|last|nth|nth-last)-(child|of-type)(?:\\("+L+"*(even|odd|(([+-]|)(\\d*)n|)"+L+"*(?:([+-]|)"+L+"*(\\d+)|))"+L+"*\\)|)","i"),bool:new RegExp("^(?:"+K+")$","i"),needsContext:new RegExp("^"+L+"*[>+~]|:(even|odd|eq|gt|lt|nth|first|last)(?:\\("+L+"*((?:-\\d)?\\d*)"+L+"*\\)|)(?=[^-]|$)","i")},X=/^(?:input|select|textarea|button)$/i,Y=/^h\d$/i,Z=/^[^{]+\{\s*\[native \w/,$=/^(?:#([\w-]+)|(\w+)|\.([\w-]+))$/,_=/[+~]/,aa=/'|\\/g,ba=new RegExp("\\\\([\\da-f]{1,6}"+L+"?|("+L+")|.)","ig"),ca=function(a,b,c){var d="0x"+b-65536;return d!==d||c?b:0>d?String.fromCharCode(d+65536):String.fromCharCode(d>>10|55296,1023&d|56320)},da=function(){m()};try{H.apply(E=I.call(v.childNodes),v.childNodes),E[v.childNodes.length].nodeType}catch(ea){H={apply:E.length?function(a,b){G.apply(a,I.call(b))}:function(a,b){var c=a.length,d=0;while(a[c++]=b[d++]);a.length=c-1}}}function fa(a,b,d,e){var f,h,j,k,l,o,r,s,w=b&&b.ownerDocument,x=b?b.nodeType:9;if(d=d||[],"string"!=typeof a||!a||1!==x&&9!==x&&11!==x)return d;if(!e&&((b?b.ownerDocument||b:v)!==n&&m(b),b=b||n,p)){if(11!==x&&(o=$.exec(a)))if(f=o[1]){if(9===x){if(!(j=b.getElementById(f)))return d;if(j.id===f)return d.push(j),d}else if(w&&(j=w.getElementById(f))&&t(b,j)&&j.id===f)return d.push(j),d}else{if(o[2])return H.apply(d,b.getElementsByTagName(a)),d;if((f=o[3])&&c.getElementsByClassName&&b.getElementsByClassName)return H.apply(d,b.getElementsByClassName(f)),d}if(c.qsa&&!A[a+" "]&&(!q||!q.test(a))){if(1!==x)w=b,s=a;else if("object"!==b.nodeName.toLowerCase()){(k=b.getAttribute("id"))?k=k.replace(aa,"\\$&"):b.setAttribute("id",k=u),r=g(a),h=r.length,l=V.test(k)?"#"+k:"[id='"+k+"']";while(h--)r[h]=l+" "+qa(r[h]);s=r.join(","),w=_.test(a)&&oa(b.parentNode)||b}if(s)try{return H.apply(d,w.querySelectorAll(s)),d}catch(y){}finally{k===u&&b.removeAttribute("id")}}}return i(a.replace(Q,"$1"),b,d,e)}function ga(){var a=[];function b(c,e){return a.push(c+" ")>d.cacheLength&&delete b[a.shift()],b[c+" "]=e}return b}function ha(a){return a[u]=!0,a}function ia(a){var b=n.createElement("div");try{return!!a(b)}catch(c){return!1}finally{b.parentNode&&b.parentNode.removeChild(b),b=null}}function ja(a,b){var c=a.split("|"),e=c.length;while(e--)d.attrHandle[c[e]]=b}function ka(a,b){var c=b&&a,d=c&&1===a.nodeType&&1===b.nodeType&&(~b.sourceIndex||C)-(~a.sourceIndex||C);if(d)return d;if(c)while(c=c.nextSibling)if(c===b)return-1;return a?1:-1}function la(a){return function(b){var c=b.nodeName.toLowerCase();return"input"===c&&b.type===a}}function ma(a){return function(b){var c=b.nodeName.toLowerCase();return("input"===c||"button"===c)&&b.type===a}}function na(a){return ha(function(b){return b=+b,ha(function(c,d){var e,f=a([],c.length,b),g=f.length;while(g--)c[e=f[g]]&&(c[e]=!(d[e]=c[e]))})})}function oa(a){return a&&"undefined"!=typeof a.getElementsByTagName&&a}c=fa.support={},f=fa.isXML=function(a){var b=a&&(a.ownerDocument||a).documentElement;return b?"HTML"!==b.nodeName:!1},m=fa.setDocument=function(a){var b,e,g=a?a.ownerDocument||a:v;return g!==n&&9===g.nodeType&&g.documentElement?(n=g,o=n.documentElement,p=!f(n),(e=n.defaultView)&&e.top!==e&&(e.addEventListener?e.addEventListener("unload",da,!1):e.attachEvent&&e.attachEvent("onunload",da)),c.attributes=ia(function(a){return a.className="i",!a.getAttribute("className")}),c.getElementsByTagName=ia(function(a){return a.appendChild(n.createComment("")),!a.getElementsByTagName("*").length}),c.getElementsByClassName=Z.test(n.getElementsByClassName),c.getById=ia(function(a){return o.appendChild(a).id=u,!n.getElementsByName||!n.getElementsByName(u).length}),c.getById?(d.find.ID=function(a,b){if("undefined"!=typeof b.getElementById&&p){var c=b.getElementById(a);return c?[c]:[]}},d.filter.ID=function(a){var b=a.replace(ba,ca);return function(a){return a.getAttribute("id")===b}}):(delete d.find.ID,d.filter.ID=function(a){var b=a.replace(ba,ca);return function(a){var c="undefined"!=typeof a.getAttributeNode&&a.getAttributeNode("id");return c&&c.value===b}}),d.find.TAG=c.getElementsByTagName?function(a,b){return"undefined"!=typeof b.getElementsByTagName?b.getElementsByTagName(a):c.qsa?b.querySelectorAll(a):void 0}:function(a,b){var c,d=[],e=0,f=b.getElementsByTagName(a);if("*"===a){while(c=f[e++])1===c.nodeType&&d.push(c);return d}return f},d.find.CLASS=c.getElementsByClassName&&function(a,b){return"undefined"!=typeof b.getElementsByClassName&&p?b.getElementsByClassName(a):void 0},r=[],q=[],(c.qsa=Z.test(n.querySelectorAll))&&(ia(function(a){o.appendChild(a).innerHTML="<a id='"+u+"'></a><select id='"+u+"-\r\\' msallowcapture=''><option selected=''></option></select>",a.querySelectorAll("[msallowcapture^='']").length&&q.push("[*^$]="+L+"*(?:''|\"\")"),a.querySelectorAll("[selected]").length||q.push("\\["+L+"*(?:value|"+K+")"),a.querySelectorAll("[id~="+u+"-]").length||q.push("~="),a.querySelectorAll(":checked").length||q.push(":checked"),a.querySelectorAll("a#"+u+"+*").length||q.push(".#.+[+~]")}),ia(function(a){var b=n.createElement("input");b.setAttribute("type","hidden"),a.appendChild(b).setAttribute("name","D"),a.querySelectorAll("[name=d]").length&&q.push("name"+L+"*[*^$|!~]?="),a.querySelectorAll(":enabled").length||q.push(":enabled",":disabled"),a.querySelectorAll("*,:x"),q.push(",.*:")})),(c.matchesSelector=Z.test(s=o.matches||o.webkitMatchesSelector||o.mozMatchesSelector||o.oMatchesSelector||o.msMatchesSelector))&&ia(function(a){c.disconnectedMatch=s.call(a,"div"),s.call(a,"[s!='']:x"),r.push("!=",O)}),q=q.length&&new RegExp(q.join("|")),r=r.length&&new RegExp(r.join("|")),b=Z.test(o.compareDocumentPosition),t=b||Z.test(o.contains)?function(a,b){var c=9===a.nodeType?a.documentElement:a,d=b&&b.parentNode;return a===d||!(!d||1!==d.nodeType||!(c.contains?c.contains(d):a.compareDocumentPosition&&16&a.compareDocumentPosition(d)))}:function(a,b){if(b)while(b=b.parentNode)if(b===a)return!0;return!1},B=b?function(a,b){if(a===b)return l=!0,0;var d=!a.compareDocumentPosition-!b.compareDocumentPosition;return d?d:(d=(a.ownerDocument||a)===(b.ownerDocument||b)?a.compareDocumentPosition(b):1,1&d||!c.sortDetached&&b.compareDocumentPosition(a)===d?a===n||a.ownerDocument===v&&t(v,a)?-1:b===n||b.ownerDocument===v&&t(v,b)?1:k?J(k,a)-J(k,b):0:4&d?-1:1)}:function(a,b){if(a===b)return l=!0,0;var c,d=0,e=a.parentNode,f=b.parentNode,g=[a],h=[b];if(!e||!f)return a===n?-1:b===n?1:e?-1:f?1:k?J(k,a)-J(k,b):0;if(e===f)return ka(a,b);c=a;while(c=c.parentNode)g.unshift(c);c=b;while(c=c.parentNode)h.unshift(c);while(g[d]===h[d])d++;return d?ka(g[d],h[d]):g[d]===v?-1:h[d]===v?1:0},n):n},fa.matches=function(a,b){return fa(a,null,null,b)},fa.matchesSelector=function(a,b){if((a.ownerDocument||a)!==n&&m(a),b=b.replace(T,"='$1']"),c.matchesSelector&&p&&!A[b+" "]&&(!r||!r.test(b))&&(!q||!q.test(b)))try{var d=s.call(a,b);if(d||c.disconnectedMatch||a.document&&11!==a.document.nodeType)return d}catch(e){}return fa(b,n,null,[a]).length>0},fa.contains=function(a,b){return(a.ownerDocument||a)!==n&&m(a),t(a,b)},fa.attr=function(a,b){(a.ownerDocument||a)!==n&&m(a);var e=d.attrHandle[b.toLowerCase()],f=e&&D.call(d.attrHandle,b.toLowerCase())?e(a,b,!p):void 0;return void 0!==f?f:c.attributes||!p?a.getAttribute(b):(f=a.getAttributeNode(b))&&f.specified?f.value:null},fa.error=function(a){throw new Error("Syntax error, unrecognized expression: "+a)},fa.uniqueSort=function(a){var b,d=[],e=0,f=0;if(l=!c.detectDuplicates,k=!c.sortStable&&a.slice(0),a.sort(B),l){while(b=a[f++])b===a[f]&&(e=d.push(f));while(e--)a.splice(d[e],1)}return k=null,a},e=fa.getText=function(a){var b,c="",d=0,f=a.nodeType;if(f){if(1===f||9===f||11===f){if("string"==typeof a.textContent)return a.textContent;for(a=a.firstChild;a;a=a.nextSibling)c+=e(a)}else if(3===f||4===f)return a.nodeValue}else while(b=a[d++])c+=e(b);return c},d=fa.selectors={cacheLength:50,createPseudo:ha,match:W,attrHandle:{},find:{},relative:{">":{dir:"parentNode",first:!0}," ":{dir:"parentNode"},"+":{dir:"previousSibling",first:!0},"~":{dir:"previousSibling"}},preFilter:{ATTR:function(a){return a[1]=a[1].replace(ba,ca),a[3]=(a[3]||a[4]||a[5]||"").replace(ba,ca),"~="===a[2]&&(a[3]=" "+a[3]+" "),a.slice(0,4)},CHILD:function(a){return a[1]=a[1].toLowerCase(),"nth"===a[1].slice(0,3)?(a[3]||fa.error(a[0]),a[4]=+(a[4]?a[5]+(a[6]||1):2*("even"===a[3]||"odd"===a[3])),a[5]=+(a[7]+a[8]||"odd"===a[3])):a[3]&&fa.error(a[0]),a},PSEUDO:function(a){var b,c=!a[6]&&a[2];return W.CHILD.test(a[0])?null:(a[3]?a[2]=a[4]||a[5]||"":c&&U.test(c)&&(b=g(c,!0))&&(b=c.indexOf(")",c.length-b)-c.length)&&(a[0]=a[0].slice(0,b),a[2]=c.slice(0,b)),a.slice(0,3))}},filter:{TAG:function(a){var b=a.replace(ba,ca).toLowerCase();return"*"===a?function(){return!0}:function(a){return a.nodeName&&a.nodeName.toLowerCase()===b}},CLASS:function(a){var b=y[a+" "];return b||(b=new RegExp("(^|"+L+")"+a+"("+L+"|$)"))&&y(a,function(a){return b.test("string"==typeof a.className&&a.className||"undefined"!=typeof a.getAttribute&&a.getAttribute("class")||"")})},ATTR:function(a,b,c){return function(d){var e=fa.attr(d,a);return null==e?"!="===b:b?(e+="","="===b?e===c:"!="===b?e!==c:"^="===b?c&&0===e.indexOf(c):"*="===b?c&&e.indexOf(c)>-1:"$="===b?c&&e.slice(-c.length)===c:"~="===b?(" "+e.replace(P," ")+" ").indexOf(c)>-1:"|="===b?e===c||e.slice(0,c.length+1)===c+"-":!1):!0}},CHILD:function(a,b,c,d,e){var f="nth"!==a.slice(0,3),g="last"!==a.slice(-4),h="of-type"===b;return 1===d&&0===e?function(a){return!!a.parentNode}:function(b,c,i){var j,k,l,m,n,o,p=f!==g?"nextSibling":"previousSibling",q=b.parentNode,r=h&&b.nodeName.toLowerCase(),s=!i&&!h,t=!1;if(q){if(f){while(p){m=b;while(m=m[p])if(h?m.nodeName.toLowerCase()===r:1===m.nodeType)return!1;o=p="only"===a&&!o&&"nextSibling"}return!0}if(o=[g?q.firstChild:q.lastChild],g&&s){m=q,l=m[u]||(m[u]={}),k=l[m.uniqueID]||(l[m.uniqueID]={}),j=k[a]||[],n=j[0]===w&&j[1],t=n&&j[2],m=n&&q.childNodes[n];while(m=++n&&m&&m[p]||(t=n=0)||o.pop())if(1===m.nodeType&&++t&&m===b){k[a]=[w,n,t];break}}else if(s&&(m=b,l=m[u]||(m[u]={}),k=l[m.uniqueID]||(l[m.uniqueID]={}),j=k[a]||[],n=j[0]===w&&j[1],t=n),t===!1)while(m=++n&&m&&m[p]||(t=n=0)||o.pop())if((h?m.nodeName.toLowerCase()===r:1===m.nodeType)&&++t&&(s&&(l=m[u]||(m[u]={}),k=l[m.uniqueID]||(l[m.uniqueID]={}),k[a]=[w,t]),m===b))break;return t-=e,t===d||t%d===0&&t/d>=0}}},PSEUDO:function(a,b){var c,e=d.pseudos[a]||d.setFilters[a.toLowerCase()]||fa.error("unsupported pseudo: "+a);return e[u]?e(b):e.length>1?(c=[a,a,"",b],d.setFilters.hasOwnProperty(a.toLowerCase())?ha(function(a,c){var d,f=e(a,b),g=f.length;while(g--)d=J(a,f[g]),a[d]=!(c[d]=f[g])}):function(a){return e(a,0,c)}):e}},pseudos:{not:ha(function(a){var b=[],c=[],d=h(a.replace(Q,"$1"));return d[u]?ha(function(a,b,c,e){var f,g=d(a,null,e,[]),h=a.length;while(h--)(f=g[h])&&(a[h]=!(b[h]=f))}):function(a,e,f){return b[0]=a,d(b,null,f,c),b[0]=null,!c.pop()}}),has:ha(function(a){return function(b){return fa(a,b).length>0}}),contains:ha(function(a){return a=a.replace(ba,ca),function(b){return(b.textContent||b.innerText||e(b)).indexOf(a)>-1}}),lang:ha(function(a){return V.test(a||"")||fa.error("unsupported lang: "+a),a=a.replace(ba,ca).toLowerCase(),function(b){var c;do if(c=p?b.lang:b.getAttribute("xml:lang")||b.getAttribute("lang"))return c=c.toLowerCase(),c===a||0===c.indexOf(a+"-");while((b=b.parentNode)&&1===b.nodeType);return!1}}),target:function(b){var c=a.location&&a.location.hash;return c&&c.slice(1)===b.id},root:function(a){return a===o},focus:function(a){return a===n.activeElement&&(!n.hasFocus||n.hasFocus())&&!!(a.type||a.href||~a.tabIndex)},enabled:function(a){return a.disabled===!1},disabled:function(a){return a.disabled===!0},checked:function(a){var b=a.nodeName.toLowerCase();return"input"===b&&!!a.checked||"option"===b&&!!a.selected},selected:function(a){return a.parentNode&&a.parentNode.selectedIndex,a.selected===!0},empty:function(a){for(a=a.firstChild;a;a=a.nextSibling)if(a.nodeType<6)return!1;return!0},parent:function(a){return!d.pseudos.empty(a)},header:function(a){return Y.test(a.nodeName)},input:function(a){return X.test(a.nodeName)},button:function(a){var b=a.nodeName.toLowerCase();return"input"===b&&"button"===a.type||"button"===b},text:function(a){var b;return"input"===a.nodeName.toLowerCase()&&"text"===a.type&&(null==(b=a.getAttribute("type"))||"text"===b.toLowerCase())},first:na(function(){return[0]}),last:na(function(a,b){return[b-1]}),eq:na(function(a,b,c){return[0>c?c+b:c]}),even:na(function(a,b){for(var c=0;b>c;c+=2)a.push(c);return a}),odd:na(function(a,b){for(var c=1;b>c;c+=2)a.push(c);return a}),lt:na(function(a,b,c){for(var d=0>c?c+b:c;--d>=0;)a.push(d);return a}),gt:na(function(a,b,c){for(var d=0>c?c+b:c;++d<b;)a.push(d);return a})}},d.pseudos.nth=d.pseudos.eq;for(b in{radio:!0,checkbox:!0,file:!0,password:!0,image:!0})d.pseudos[b]=la(b);for(b in{submit:!0,reset:!0})d.pseudos[b]=ma(b);function pa(){}pa.prototype=d.filters=d.pseudos,d.setFilters=new pa,g=fa.tokenize=function(a,b){var c,e,f,g,h,i,j,k=z[a+" "];if(k)return b?0:k.slice(0);h=a,i=[],j=d.preFilter;while(h){c&&!(e=R.exec(h))||(e&&(h=h.slice(e[0].length)||h),i.push(f=[])),c=!1,(e=S.exec(h))&&(c=e.shift(),f.push({value:c,type:e[0].replace(Q," ")}),h=h.slice(c.length));for(g in d.filter)!(e=W[g].exec(h))||j[g]&&!(e=j[g](e))||(c=e.shift(),f.push({value:c,type:g,matches:e}),h=h.slice(c.length));if(!c)break}return b?h.length:h?fa.error(a):z(a,i).slice(0)};function qa(a){for(var b=0,c=a.length,d="";c>b;b++)d+=a[b].value;return d}function ra(a,b,c){var d=b.dir,e=c&&"parentNode"===d,f=x++;return b.first?function(b,c,f){while(b=b[d])if(1===b.nodeType||e)return a(b,c,f)}:function(b,c,g){var h,i,j,k=[w,f];if(g){while(b=b[d])if((1===b.nodeType||e)&&a(b,c,g))return!0}else while(b=b[d])if(1===b.nodeType||e){if(j=b[u]||(b[u]={}),i=j[b.uniqueID]||(j[b.uniqueID]={}),(h=i[d])&&h[0]===w&&h[1]===f)return k[2]=h[2];if(i[d]=k,k[2]=a(b,c,g))return!0}}}function sa(a){return a.length>1?function(b,c,d){var e=a.length;while(e--)if(!a[e](b,c,d))return!1;return!0}:a[0]}function ta(a,b,c){for(var d=0,e=b.length;e>d;d++)fa(a,b[d],c);return c}function ua(a,b,c,d,e){for(var f,g=[],h=0,i=a.length,j=null!=b;i>h;h++)(f=a[h])&&(c&&!c(f,d,e)||(g.push(f),j&&b.push(h)));return g}function va(a,b,c,d,e,f){return d&&!d[u]&&(d=va(d)),e&&!e[u]&&(e=va(e,f)),ha(function(f,g,h,i){var j,k,l,m=[],n=[],o=g.length,p=f||ta(b||"*",h.nodeType?[h]:h,[]),q=!a||!f&&b?p:ua(p,m,a,h,i),r=c?e||(f?a:o||d)?[]:g:q;if(c&&c(q,r,h,i),d){j=ua(r,n),d(j,[],h,i),k=j.length;while(k--)(l=j[k])&&(r[n[k]]=!(q[n[k]]=l))}if(f){if(e||a){if(e){j=[],k=r.length;while(k--)(l=r[k])&&j.push(q[k]=l);e(null,r=[],j,i)}k=r.length;while(k--)(l=r[k])&&(j=e?J(f,l):m[k])>-1&&(f[j]=!(g[j]=l))}}else r=ua(r===g?r.splice(o,r.length):r),e?e(null,g,r,i):H.apply(g,r)})}function wa(a){for(var b,c,e,f=a.length,g=d.relative[a[0].type],h=g||d.relative[" "],i=g?1:0,k=ra(function(a){return a===b},h,!0),l=ra(function(a){return J(b,a)>-1},h,!0),m=[function(a,c,d){var e=!g&&(d||c!==j)||((b=c).nodeType?k(a,c,d):l(a,c,d));return b=null,e}];f>i;i++)if(c=d.relative[a[i].type])m=[ra(sa(m),c)];else{if(c=d.filter[a[i].type].apply(null,a[i].matches),c[u]){for(e=++i;f>e;e++)if(d.relative[a[e].type])break;return va(i>1&&sa(m),i>1&&qa(a.slice(0,i-1).concat({value:" "===a[i-2].type?"*":""})).replace(Q,"$1"),c,e>i&&wa(a.slice(i,e)),f>e&&wa(a=a.slice(e)),f>e&&qa(a))}m.push(c)}return sa(m)}function xa(a,b){var c=b.length>0,e=a.length>0,f=function(f,g,h,i,k){var l,o,q,r=0,s="0",t=f&&[],u=[],v=j,x=f||e&&d.find.TAG("*",k),y=w+=null==v?1:Math.random()||.1,z=x.length;for(k&&(j=g===n||g||k);s!==z&&null!=(l=x[s]);s++){if(e&&l){o=0,g||l.ownerDocument===n||(m(l),h=!p);while(q=a[o++])if(q(l,g||n,h)){i.push(l);break}k&&(w=y)}c&&((l=!q&&l)&&r--,f&&t.push(l))}if(r+=s,c&&s!==r){o=0;while(q=b[o++])q(t,u,g,h);if(f){if(r>0)while(s--)t[s]||u[s]||(u[s]=F.call(i));u=ua(u)}H.apply(i,u),k&&!f&&u.length>0&&r+b.length>1&&fa.uniqueSort(i)}return k&&(w=y,j=v),t};return c?ha(f):f}return h=fa.compile=function(a,b){var c,d=[],e=[],f=A[a+" "];if(!f){b||(b=g(a)),c=b.length;while(c--)f=wa(b[c]),f[u]?d.push(f):e.push(f);f=A(a,xa(e,d)),f.selector=a}return f},i=fa.select=function(a,b,e,f){var i,j,k,l,m,n="function"==typeof a&&a,o=!f&&g(a=n.selector||a);if(e=e||[],1===o.length){if(j=o[0]=o[0].slice(0),j.length>2&&"ID"===(k=j[0]).type&&c.getById&&9===b.nodeType&&p&&d.relative[j[1].type]){if(b=(d.find.ID(k.matches[0].replace(ba,ca),b)||[])[0],!b)return e;n&&(b=b.parentNode),a=a.slice(j.shift().value.length)}i=W.needsContext.test(a)?0:j.length;while(i--){if(k=j[i],d.relative[l=k.type])break;if((m=d.find[l])&&(f=m(k.matches[0].replace(ba,ca),_.test(j[0].type)&&oa(b.parentNode)||b))){if(j.splice(i,1),a=f.length&&qa(j),!a)return H.apply(e,f),e;break}}}return(n||h(a,o))(f,b,!p,e,!b||_.test(a)&&oa(b.parentNode)||b),e},c.sortStable=u.split("").sort(B).join("")===u,c.detectDuplicates=!!l,m(),c.sortDetached=ia(function(a){return 1&a.compareDocumentPosition(n.createElement("div"))}),ia(function(a){return a.innerHTML="<a href='#'></a>","#"===a.firstChild.getAttribute("href")})||ja("type|href|height|width",function(a,b,c){return c?void 0:a.getAttribute(b,"type"===b.toLowerCase()?1:2)}),c.attributes&&ia(function(a){return a.innerHTML="<input/>",a.firstChild.setAttribute("value",""),""===a.firstChild.getAttribute("value")})||ja("value",function(a,b,c){return c||"input"!==a.nodeName.toLowerCase()?void 0:a.defaultValue}),ia(function(a){return null==a.getAttribute("disabled")})||ja(K,function(a,b,c){var d;return c?void 0:a[b]===!0?b.toLowerCase():(d=a.getAttributeNode(b))&&d.specified?d.value:null}),fa}(a);n.find=t,n.expr=t.selectors,n.expr[":"]=n.expr.pseudos,n.uniqueSort=n.unique=t.uniqueSort,n.text=t.getText,n.isXMLDoc=t.isXML,n.contains=t.contains;var u=function(a,b,c){var d=[],e=void 0!==c;while((a=a[b])&&9!==a.nodeType)if(1===a.nodeType){if(e&&n(a).is(c))break;d.push(a)}return d},v=function(a,b){for(var c=[];a;a=a.nextSibling)1===a.nodeType&&a!==b&&c.push(a);return c},w=n.expr.match.needsContext,x=/^<([\w-]+)\s*\/?>(?:<\/\1>|)$/,y=/^.[^:#\[\.,]*$/;function z(a,b,c){if(n.isFunction(b))return n.grep(a,function(a,d){return!!b.call(a,d,a)!==c});if(b.nodeType)return n.grep(a,function(a){return a===b!==c});if("string"==typeof b){if(y.test(b))return n.filter(b,a,c);b=n.filter(b,a)}return n.grep(a,function(a){return n.inArray(a,b)>-1!==c})}n.filter=function(a,b,c){var d=b[0];return c&&(a=":not("+a+")"),1===b.length&&1===d.nodeType?n.find.matchesSelector(d,a)?[d]:[]:n.find.matches(a,n.grep(b,function(a){return 1===a.nodeType}))},n.fn.extend({find:function(a){var b,c=[],d=this,e=d.length;if("string"!=typeof a)return this.pushStack(n(a).filter(function(){for(b=0;e>b;b++)if(n.contains(d[b],this))return!0}));for(b=0;e>b;b++)n.find(a,d[b],c);return c=this.pushStack(e>1?n.unique(c):c),c.selector=this.selector?this.selector+" "+a:a,c},filter:function(a){return this.pushStack(z(this,a||[],!1))},not:function(a){return this.pushStack(z(this,a||[],!0))},is:function(a){return!!z(this,"string"==typeof a&&w.test(a)?n(a):a||[],!1).length}});var A,B=/^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/,C=n.fn.init=function(a,b,c){var e,f;if(!a)return this;if(c=c||A,"string"==typeof a){if(e="<"===a.charAt(0)&&">"===a.charAt(a.length-1)&&a.length>=3?[null,a,null]:B.exec(a),!e||!e[1]&&b)return!b||b.jquery?(b||c).find(a):this.constructor(b).find(a);if(e[1]){if(b=b instanceof n?b[0]:b,n.merge(this,n.parseHTML(e[1],b&&b.nodeType?b.ownerDocument||b:d,!0)),x.test(e[1])&&n.isPlainObject(b))for(e in b)n.isFunction(this[e])?this[e](b[e]):this.attr(e,b[e]);return this}if(f=d.getElementById(e[2]),f&&f.parentNode){if(f.id!==e[2])return A.find(a);this.length=1,this[0]=f}return this.context=d,this.selector=a,this}return a.nodeType?(this.context=this[0]=a,this.length=1,this):n.isFunction(a)?"undefined"!=typeof c.ready?c.ready(a):a(n):(void 0!==a.selector&&(this.selector=a.selector,this.context=a.context),n.makeArray(a,this))};C.prototype=n.fn,A=n(d);var D=/^(?:parents|prev(?:Until|All))/,E={children:!0,contents:!0,next:!0,prev:!0};n.fn.extend({has:function(a){var b,c=n(a,this),d=c.length;return this.filter(function(){for(b=0;d>b;b++)if(n.contains(this,c[b]))return!0})},closest:function(a,b){for(var c,d=0,e=this.length,f=[],g=w.test(a)||"string"!=typeof a?n(a,b||this.context):0;e>d;d++)for(c=this[d];c&&c!==b;c=c.parentNode)if(c.nodeType<11&&(g?g.index(c)>-1:1===c.nodeType&&n.find.matchesSelector(c,a))){f.push(c);break}return this.pushStack(f.length>1?n.uniqueSort(f):f)},index:function(a){return a?"string"==typeof a?n.inArray(this[0],n(a)):n.inArray(a.jquery?a[0]:a,this):this[0]&&this[0].parentNode?this.first().prevAll().length:-1},add:function(a,b){return this.pushStack(n.uniqueSort(n.merge(this.get(),n(a,b))))},addBack:function(a){return this.add(null==a?this.prevObject:this.prevObject.filter(a))}});function F(a,b){do a=a[b];while(a&&1!==a.nodeType);return a}n.each({parent:function(a){var b=a.parentNode;return b&&11!==b.nodeType?b:null},parents:function(a){return u(a,"parentNode")},parentsUntil:function(a,b,c){return u(a,"parentNode",c)},next:function(a){return F(a,"nextSibling")},prev:function(a){return F(a,"previousSibling")},nextAll:function(a){return u(a,"nextSibling")},prevAll:function(a){return u(a,"previousSibling")},nextUntil:function(a,b,c){return u(a,"nextSibling",c)},prevUntil:function(a,b,c){return u(a,"previousSibling",c)},siblings:function(a){return v((a.parentNode||{}).firstChild,a)},children:function(a){return v(a.firstChild)},contents:function(a){return n.nodeName(a,"iframe")?a.contentDocument||a.contentWindow.document:n.merge([],a.childNodes)}},function(a,b){n.fn[a]=function(c,d){var e=n.map(this,b,c);return"Until"!==a.slice(-5)&&(d=c),d&&"string"==typeof d&&(e=n.filter(d,e)),this.length>1&&(E[a]||(e=n.uniqueSort(e)),D.test(a)&&(e=e.reverse())),this.pushStack(e)}});var G=/\S+/g;function H(a){var b={};return n.each(a.match(G)||[],function(a,c){b[c]=!0}),b}n.Callbacks=function(a){a="string"==typeof a?H(a):n.extend({},a);var b,c,d,e,f=[],g=[],h=-1,i=function(){for(e=a.once,d=b=!0;g.length;h=-1){c=g.shift();while(++h<f.length)f[h].apply(c[0],c[1])===!1&&a.stopOnFalse&&(h=f.length,c=!1)}a.memory||(c=!1),b=!1,e&&(f=c?[]:"")},j={add:function(){return f&&(c&&!b&&(h=f.length-1,g.push(c)),function d(b){n.each(b,function(b,c){n.isFunction(c)?a.unique&&j.has(c)||f.push(c):c&&c.length&&"string"!==n.type(c)&&d(c)})}(arguments),c&&!b&&i()),this},remove:function(){return n.each(arguments,function(a,b){var c;while((c=n.inArray(b,f,c))>-1)f.splice(c,1),h>=c&&h--}),this},has:function(a){return a?n.inArray(a,f)>-1:f.length>0},empty:function(){return f&&(f=[]),this},disable:function(){return e=g=[],f=c="",this},disabled:function(){return!f},lock:function(){return e=!0,c||j.disable(),this},locked:function(){return!!e},fireWith:function(a,c){return e||(c=c||[],c=[a,c.slice?c.slice():c],g.push(c),b||i()),this},fire:function(){return j.fireWith(this,arguments),this},fired:function(){return!!d}};return j},n.extend({Deferred:function(a){var b=[["resolve","done",n.Callbacks("once memory"),"resolved"],["reject","fail",n.Callbacks("once memory"),"rejected"],["notify","progress",n.Callbacks("memory")]],c="pending",d={state:function(){return c},always:function(){return e.done(arguments).fail(arguments),this},then:function(){var a=arguments;return n.Deferred(function(c){n.each(b,function(b,f){var g=n.isFunction(a[b])&&a[b];e[f[1]](function(){var a=g&&g.apply(this,arguments);a&&n.isFunction(a.promise)?a.promise().progress(c.notify).done(c.resolve).fail(c.reject):c[f[0]+"With"](this===d?c.promise():this,g?[a]:arguments)})}),a=null}).promise()},promise:function(a){return null!=a?n.extend(a,d):d}},e={};return d.pipe=d.then,n.each(b,function(a,f){var g=f[2],h=f[3];d[f[1]]=g.add,h&&g.add(function(){c=h},b[1^a][2].disable,b[2][2].lock),e[f[0]]=function(){return e[f[0]+"With"](this===e?d:this,arguments),this},e[f[0]+"With"]=g.fireWith}),d.promise(e),a&&a.call(e,e),e},when:function(a){var b=0,c=e.call(arguments),d=c.length,f=1!==d||a&&n.isFunction(a.promise)?d:0,g=1===f?a:n.Deferred(),h=function(a,b,c){return function(d){b[a]=this,c[a]=arguments.length>1?e.call(arguments):d,c===i?g.notifyWith(b,c):--f||g.resolveWith(b,c)}},i,j,k;if(d>1)for(i=new Array(d),j=new Array(d),k=new Array(d);d>b;b++)c[b]&&n.isFunction(c[b].promise)?c[b].promise().progress(h(b,j,i)).done(h(b,k,c)).fail(g.reject):--f;return f||g.resolveWith(k,c),g.promise()}});var I;n.fn.ready=function(a){return n.ready.promise().done(a),this},n.extend({isReady:!1,readyWait:1,holdReady:function(a){a?n.readyWait++:n.ready(!0)},ready:function(a){(a===!0?--n.readyWait:n.isReady)||(n.isReady=!0,a!==!0&&--n.readyWait>0||(I.resolveWith(d,[n]),n.fn.triggerHandler&&(n(d).triggerHandler("ready"),n(d).off("ready"))))}});function J(){d.addEventListener?(d.removeEventListener("DOMContentLoaded",K),a.removeEventListener("load",K)):(d.detachEvent("onreadystatechange",K),a.detachEvent("onload",K))}function K(){(d.addEventListener||"load"===a.event.type||"complete"===d.readyState)&&(J(),n.ready())}n.ready.promise=function(b){if(!I)if(I=n.Deferred(),"complete"===d.readyState||"loading"!==d.readyState&&!d.documentElement.doScroll)a.setTimeout(n.ready);else if(d.addEventListener)d.addEventListener("DOMContentLoaded",K),a.addEventListener("load",K);else{d.attachEvent("onreadystatechange",K),a.attachEvent("onload",K);var c=!1;try{c=null==a.frameElement&&d.documentElement}catch(e){}c&&c.doScroll&&!function f(){if(!n.isReady){try{c.doScroll("left")}catch(b){return a.setTimeout(f,50)}J(),n.ready()}}()}return I.promise(b)},n.ready.promise();var L;for(L in n(l))break;l.ownFirst="0"===L,l.inlineBlockNeedsLayout=!1,n(function(){var a,b,c,e;c=d.getElementsByTagName("body")[0],c&&c.style&&(b=d.createElement("div"),e=d.createElement("div"),e.style.cssText="position:absolute;border:0;width:0;height:0;top:0;left:-9999px",c.appendChild(e).appendChild(b),"undefined"!=typeof b.style.zoom&&(b.style.cssText="display:inline;margin:0;border:0;padding:1px;width:1px;zoom:1",l.inlineBlockNeedsLayout=a=3===b.offsetWidth,a&&(c.style.zoom=1)),c.removeChild(e))}),function(){var a=d.createElement("div");l.deleteExpando=!0;try{delete a.test}catch(b){l.deleteExpando=!1}a=null}();var M=function(a){var b=n.noData[(a.nodeName+" ").toLowerCase()],c=+a.nodeType||1;return 1!==c&&9!==c?!1:!b||b!==!0&&a.getAttribute("classid")===b},N=/^(?:\{[\w\W]*\}|\[[\w\W]*\])$/,O=/([A-Z])/g;function P(a,b,c){if(void 0===c&&1===a.nodeType){var d="data-"+b.replace(O,"-$1").toLowerCase();if(c=a.getAttribute(d),"string"==typeof c){try{c="true"===c?!0:"false"===c?!1:"null"===c?null:+c+""===c?+c:N.test(c)?n.parseJSON(c):c}catch(e){}n.data(a,b,c)}else c=void 0;
}return c}function Q(a){var b;for(b in a)if(("data"!==b||!n.isEmptyObject(a[b]))&&"toJSON"!==b)return!1;return!0}function R(a,b,d,e){if(M(a)){var f,g,h=n.expando,i=a.nodeType,j=i?n.cache:a,k=i?a[h]:a[h]&&h;if(k&&j[k]&&(e||j[k].data)||void 0!==d||"string"!=typeof b)return k||(k=i?a[h]=c.pop()||n.guid++:h),j[k]||(j[k]=i?{}:{toJSON:n.noop}),"object"!=typeof b&&"function"!=typeof b||(e?j[k]=n.extend(j[k],b):j[k].data=n.extend(j[k].data,b)),g=j[k],e||(g.data||(g.data={}),g=g.data),void 0!==d&&(g[n.camelCase(b)]=d),"string"==typeof b?(f=g[b],null==f&&(f=g[n.camelCase(b)])):f=g,f}}function S(a,b,c){if(M(a)){var d,e,f=a.nodeType,g=f?n.cache:a,h=f?a[n.expando]:n.expando;if(g[h]){if(b&&(d=c?g[h]:g[h].data)){n.isArray(b)?b=b.concat(n.map(b,n.camelCase)):b in d?b=[b]:(b=n.camelCase(b),b=b in d?[b]:b.split(" ")),e=b.length;while(e--)delete d[b[e]];if(c?!Q(d):!n.isEmptyObject(d))return}(c||(delete g[h].data,Q(g[h])))&&(f?n.cleanData([a],!0):l.deleteExpando||g!=g.window?delete g[h]:g[h]=void 0)}}}n.extend({cache:{},noData:{"applet ":!0,"embed ":!0,"object ":"clsid:D27CDB6E-AE6D-11cf-96B8-444553540000"},hasData:function(a){return a=a.nodeType?n.cache[a[n.expando]]:a[n.expando],!!a&&!Q(a)},data:function(a,b,c){return R(a,b,c)},removeData:function(a,b){return S(a,b)},_data:function(a,b,c){return R(a,b,c,!0)},_removeData:function(a,b){return S(a,b,!0)}}),n.fn.extend({data:function(a,b){var c,d,e,f=this[0],g=f&&f.attributes;if(void 0===a){if(this.length&&(e=n.data(f),1===f.nodeType&&!n._data(f,"parsedAttrs"))){c=g.length;while(c--)g[c]&&(d=g[c].name,0===d.indexOf("data-")&&(d=n.camelCase(d.slice(5)),P(f,d,e[d])));n._data(f,"parsedAttrs",!0)}return e}return"object"==typeof a?this.each(function(){n.data(this,a)}):arguments.length>1?this.each(function(){n.data(this,a,b)}):f?P(f,a,n.data(f,a)):void 0},removeData:function(a){return this.each(function(){n.removeData(this,a)})}}),n.extend({queue:function(a,b,c){var d;return a?(b=(b||"fx")+"queue",d=n._data(a,b),c&&(!d||n.isArray(c)?d=n._data(a,b,n.makeArray(c)):d.push(c)),d||[]):void 0},dequeue:function(a,b){b=b||"fx";var c=n.queue(a,b),d=c.length,e=c.shift(),f=n._queueHooks(a,b),g=function(){n.dequeue(a,b)};"inprogress"===e&&(e=c.shift(),d--),e&&("fx"===b&&c.unshift("inprogress"),delete f.stop,e.call(a,g,f)),!d&&f&&f.empty.fire()},_queueHooks:function(a,b){var c=b+"queueHooks";return n._data(a,c)||n._data(a,c,{empty:n.Callbacks("once memory").add(function(){n._removeData(a,b+"queue"),n._removeData(a,c)})})}}),n.fn.extend({queue:function(a,b){var c=2;return"string"!=typeof a&&(b=a,a="fx",c--),arguments.length<c?n.queue(this[0],a):void 0===b?this:this.each(function(){var c=n.queue(this,a,b);n._queueHooks(this,a),"fx"===a&&"inprogress"!==c[0]&&n.dequeue(this,a)})},dequeue:function(a){return this.each(function(){n.dequeue(this,a)})},clearQueue:function(a){return this.queue(a||"fx",[])},promise:function(a,b){var c,d=1,e=n.Deferred(),f=this,g=this.length,h=function(){--d||e.resolveWith(f,[f])};"string"!=typeof a&&(b=a,a=void 0),a=a||"fx";while(g--)c=n._data(f[g],a+"queueHooks"),c&&c.empty&&(d++,c.empty.add(h));return h(),e.promise(b)}}),function(){var a;l.shrinkWrapBlocks=function(){if(null!=a)return a;a=!1;var b,c,e;return c=d.getElementsByTagName("body")[0],c&&c.style?(b=d.createElement("div"),e=d.createElement("div"),e.style.cssText="position:absolute;border:0;width:0;height:0;top:0;left:-9999px",c.appendChild(e).appendChild(b),"undefined"!=typeof b.style.zoom&&(b.style.cssText="-webkit-box-sizing:content-box;-moz-box-sizing:content-box;box-sizing:content-box;display:block;margin:0;border:0;padding:1px;width:1px;zoom:1",b.appendChild(d.createElement("div")).style.width="5px",a=3!==b.offsetWidth),c.removeChild(e),a):void 0}}();var T=/[+-]?(?:\d*\.|)\d+(?:[eE][+-]?\d+|)/.source,U=new RegExp("^(?:([+-])=|)("+T+")([a-z%]*)$","i"),V=["Top","Right","Bottom","Left"],W=function(a,b){return a=b||a,"none"===n.css(a,"display")||!n.contains(a.ownerDocument,a)};function X(a,b,c,d){var e,f=1,g=20,h=d?function(){return d.cur()}:function(){return n.css(a,b,"")},i=h(),j=c&&c[3]||(n.cssNumber[b]?"":"px"),k=(n.cssNumber[b]||"px"!==j&&+i)&&U.exec(n.css(a,b));if(k&&k[3]!==j){j=j||k[3],c=c||[],k=+i||1;do f=f||".5",k/=f,n.style(a,b,k+j);while(f!==(f=h()/i)&&1!==f&&--g)}return c&&(k=+k||+i||0,e=c[1]?k+(c[1]+1)*c[2]:+c[2],d&&(d.unit=j,d.start=k,d.end=e)),e}var Y=function(a,b,c,d,e,f,g){var h=0,i=a.length,j=null==c;if("object"===n.type(c)){e=!0;for(h in c)Y(a,b,h,c[h],!0,f,g)}else if(void 0!==d&&(e=!0,n.isFunction(d)||(g=!0),j&&(g?(b.call(a,d),b=null):(j=b,b=function(a,b,c){return j.call(n(a),c)})),b))for(;i>h;h++)b(a[h],c,g?d:d.call(a[h],h,b(a[h],c)));return e?a:j?b.call(a):i?b(a[0],c):f},Z=/^(?:checkbox|radio)$/i,$=/<([\w:-]+)/,_=/^$|\/(?:java|ecma)script/i,aa=/^\s+/,ba="abbr|article|aside|audio|bdi|canvas|data|datalist|details|dialog|figcaption|figure|footer|header|hgroup|main|mark|meter|nav|output|picture|progress|section|summary|template|time|video";function ca(a){var b=ba.split("|"),c=a.createDocumentFragment();if(c.createElement)while(b.length)c.createElement(b.pop());return c}!function(){var a=d.createElement("div"),b=d.createDocumentFragment(),c=d.createElement("input");a.innerHTML="  <link/><table></table><a href='/a'>a</a><input type='checkbox'/>",l.leadingWhitespace=3===a.firstChild.nodeType,l.tbody=!a.getElementsByTagName("tbody").length,l.htmlSerialize=!!a.getElementsByTagName("link").length,l.html5Clone="<:nav></:nav>"!==d.createElement("nav").cloneNode(!0).outerHTML,c.type="checkbox",c.checked=!0,b.appendChild(c),l.appendChecked=c.checked,a.innerHTML="<textarea>x</textarea>",l.noCloneChecked=!!a.cloneNode(!0).lastChild.defaultValue,b.appendChild(a),c=d.createElement("input"),c.setAttribute("type","radio"),c.setAttribute("checked","checked"),c.setAttribute("name","t"),a.appendChild(c),l.checkClone=a.cloneNode(!0).cloneNode(!0).lastChild.checked,l.noCloneEvent=!!a.addEventListener,a[n.expando]=1,l.attributes=!a.getAttribute(n.expando)}();var da={option:[1,"<select multiple='multiple'>","</select>"],legend:[1,"<fieldset>","</fieldset>"],area:[1,"<map>","</map>"],param:[1,"<object>","</object>"],thead:[1,"<table>","</table>"],tr:[2,"<table><tbody>","</tbody></table>"],col:[2,"<table><tbody></tbody><colgroup>","</colgroup></table>"],td:[3,"<table><tbody><tr>","</tr></tbody></table>"],_default:l.htmlSerialize?[0,"",""]:[1,"X<div>","</div>"]};da.optgroup=da.option,da.tbody=da.tfoot=da.colgroup=da.caption=da.thead,da.th=da.td;function ea(a,b){var c,d,e=0,f="undefined"!=typeof a.getElementsByTagName?a.getElementsByTagName(b||"*"):"undefined"!=typeof a.querySelectorAll?a.querySelectorAll(b||"*"):void 0;if(!f)for(f=[],c=a.childNodes||a;null!=(d=c[e]);e++)!b||n.nodeName(d,b)?f.push(d):n.merge(f,ea(d,b));return void 0===b||b&&n.nodeName(a,b)?n.merge([a],f):f}function fa(a,b){for(var c,d=0;null!=(c=a[d]);d++)n._data(c,"globalEval",!b||n._data(b[d],"globalEval"))}var ga=/<|&#?\w+;/,ha=/<tbody/i;function ia(a){Z.test(a.type)&&(a.defaultChecked=a.checked)}function ja(a,b,c,d,e){for(var f,g,h,i,j,k,m,o=a.length,p=ca(b),q=[],r=0;o>r;r++)if(g=a[r],g||0===g)if("object"===n.type(g))n.merge(q,g.nodeType?[g]:g);else if(ga.test(g)){i=i||p.appendChild(b.createElement("div")),j=($.exec(g)||["",""])[1].toLowerCase(),m=da[j]||da._default,i.innerHTML=m[1]+n.htmlPrefilter(g)+m[2],f=m[0];while(f--)i=i.lastChild;if(!l.leadingWhitespace&&aa.test(g)&&q.push(b.createTextNode(aa.exec(g)[0])),!l.tbody){g="table"!==j||ha.test(g)?"<table>"!==m[1]||ha.test(g)?0:i:i.firstChild,f=g&&g.childNodes.length;while(f--)n.nodeName(k=g.childNodes[f],"tbody")&&!k.childNodes.length&&g.removeChild(k)}n.merge(q,i.childNodes),i.textContent="";while(i.firstChild)i.removeChild(i.firstChild);i=p.lastChild}else q.push(b.createTextNode(g));i&&p.removeChild(i),l.appendChecked||n.grep(ea(q,"input"),ia),r=0;while(g=q[r++])if(d&&n.inArray(g,d)>-1)e&&e.push(g);else if(h=n.contains(g.ownerDocument,g),i=ea(p.appendChild(g),"script"),h&&fa(i),c){f=0;while(g=i[f++])_.test(g.type||"")&&c.push(g)}return i=null,p}!function(){var b,c,e=d.createElement("div");for(b in{submit:!0,change:!0,focusin:!0})c="on"+b,(l[b]=c in a)||(e.setAttribute(c,"t"),l[b]=e.attributes[c].expando===!1);e=null}();var ka=/^(?:input|select|textarea)$/i,la=/^key/,ma=/^(?:mouse|pointer|contextmenu|drag|drop)|click/,na=/^(?:focusinfocus|focusoutblur)$/,oa=/^([^.]*)(?:\.(.+)|)/;function pa(){return!0}function qa(){return!1}function ra(){try{return d.activeElement}catch(a){}}function sa(a,b,c,d,e,f){var g,h;if("object"==typeof b){"string"!=typeof c&&(d=d||c,c=void 0);for(h in b)sa(a,h,c,d,b[h],f);return a}if(null==d&&null==e?(e=c,d=c=void 0):null==e&&("string"==typeof c?(e=d,d=void 0):(e=d,d=c,c=void 0)),e===!1)e=qa;else if(!e)return a;return 1===f&&(g=e,e=function(a){return n().off(a),g.apply(this,arguments)},e.guid=g.guid||(g.guid=n.guid++)),a.each(function(){n.event.add(this,b,e,d,c)})}n.event={global:{},add:function(a,b,c,d,e){var f,g,h,i,j,k,l,m,o,p,q,r=n._data(a);if(r){c.handler&&(i=c,c=i.handler,e=i.selector),c.guid||(c.guid=n.guid++),(g=r.events)||(g=r.events={}),(k=r.handle)||(k=r.handle=function(a){return"undefined"==typeof n||a&&n.event.triggered===a.type?void 0:n.event.dispatch.apply(k.elem,arguments)},k.elem=a),b=(b||"").match(G)||[""],h=b.length;while(h--)f=oa.exec(b[h])||[],o=q=f[1],p=(f[2]||"").split(".").sort(),o&&(j=n.event.special[o]||{},o=(e?j.delegateType:j.bindType)||o,j=n.event.special[o]||{},l=n.extend({type:o,origType:q,data:d,handler:c,guid:c.guid,selector:e,needsContext:e&&n.expr.match.needsContext.test(e),namespace:p.join(".")},i),(m=g[o])||(m=g[o]=[],m.delegateCount=0,j.setup&&j.setup.call(a,d,p,k)!==!1||(a.addEventListener?a.addEventListener(o,k,!1):a.attachEvent&&a.attachEvent("on"+o,k))),j.add&&(j.add.call(a,l),l.handler.guid||(l.handler.guid=c.guid)),e?m.splice(m.delegateCount++,0,l):m.push(l),n.event.global[o]=!0);a=null}},remove:function(a,b,c,d,e){var f,g,h,i,j,k,l,m,o,p,q,r=n.hasData(a)&&n._data(a);if(r&&(k=r.events)){b=(b||"").match(G)||[""],j=b.length;while(j--)if(h=oa.exec(b[j])||[],o=q=h[1],p=(h[2]||"").split(".").sort(),o){l=n.event.special[o]||{},o=(d?l.delegateType:l.bindType)||o,m=k[o]||[],h=h[2]&&new RegExp("(^|\\.)"+p.join("\\.(?:.*\\.|)")+"(\\.|$)"),i=f=m.length;while(f--)g=m[f],!e&&q!==g.origType||c&&c.guid!==g.guid||h&&!h.test(g.namespace)||d&&d!==g.selector&&("**"!==d||!g.selector)||(m.splice(f,1),g.selector&&m.delegateCount--,l.remove&&l.remove.call(a,g));i&&!m.length&&(l.teardown&&l.teardown.call(a,p,r.handle)!==!1||n.removeEvent(a,o,r.handle),delete k[o])}else for(o in k)n.event.remove(a,o+b[j],c,d,!0);n.isEmptyObject(k)&&(delete r.handle,n._removeData(a,"events"))}},trigger:function(b,c,e,f){var g,h,i,j,l,m,o,p=[e||d],q=k.call(b,"type")?b.type:b,r=k.call(b,"namespace")?b.namespace.split("."):[];if(i=m=e=e||d,3!==e.nodeType&&8!==e.nodeType&&!na.test(q+n.event.triggered)&&(q.indexOf(".")>-1&&(r=q.split("."),q=r.shift(),r.sort()),h=q.indexOf(":")<0&&"on"+q,b=b[n.expando]?b:new n.Event(q,"object"==typeof b&&b),b.isTrigger=f?2:3,b.namespace=r.join("."),b.rnamespace=b.namespace?new RegExp("(^|\\.)"+r.join("\\.(?:.*\\.|)")+"(\\.|$)"):null,b.result=void 0,b.target||(b.target=e),c=null==c?[b]:n.makeArray(c,[b]),l=n.event.special[q]||{},f||!l.trigger||l.trigger.apply(e,c)!==!1)){if(!f&&!l.noBubble&&!n.isWindow(e)){for(j=l.delegateType||q,na.test(j+q)||(i=i.parentNode);i;i=i.parentNode)p.push(i),m=i;m===(e.ownerDocument||d)&&p.push(m.defaultView||m.parentWindow||a)}o=0;while((i=p[o++])&&!b.isPropagationStopped())b.type=o>1?j:l.bindType||q,g=(n._data(i,"events")||{})[b.type]&&n._data(i,"handle"),g&&g.apply(i,c),g=h&&i[h],g&&g.apply&&M(i)&&(b.result=g.apply(i,c),b.result===!1&&b.preventDefault());if(b.type=q,!f&&!b.isDefaultPrevented()&&(!l._default||l._default.apply(p.pop(),c)===!1)&&M(e)&&h&&e[q]&&!n.isWindow(e)){m=e[h],m&&(e[h]=null),n.event.triggered=q;try{e[q]()}catch(s){}n.event.triggered=void 0,m&&(e[h]=m)}return b.result}},dispatch:function(a){a=n.event.fix(a);var b,c,d,f,g,h=[],i=e.call(arguments),j=(n._data(this,"events")||{})[a.type]||[],k=n.event.special[a.type]||{};if(i[0]=a,a.delegateTarget=this,!k.preDispatch||k.preDispatch.call(this,a)!==!1){h=n.event.handlers.call(this,a,j),b=0;while((f=h[b++])&&!a.isPropagationStopped()){a.currentTarget=f.elem,c=0;while((g=f.handlers[c++])&&!a.isImmediatePropagationStopped())a.rnamespace&&!a.rnamespace.test(g.namespace)||(a.handleObj=g,a.data=g.data,d=((n.event.special[g.origType]||{}).handle||g.handler).apply(f.elem,i),void 0!==d&&(a.result=d)===!1&&(a.preventDefault(),a.stopPropagation()))}return k.postDispatch&&k.postDispatch.call(this,a),a.result}},handlers:function(a,b){var c,d,e,f,g=[],h=b.delegateCount,i=a.target;if(h&&i.nodeType&&("click"!==a.type||isNaN(a.button)||a.button<1))for(;i!=this;i=i.parentNode||this)if(1===i.nodeType&&(i.disabled!==!0||"click"!==a.type)){for(d=[],c=0;h>c;c++)f=b[c],e=f.selector+" ",void 0===d[e]&&(d[e]=f.needsContext?n(e,this).index(i)>-1:n.find(e,this,null,[i]).length),d[e]&&d.push(f);d.length&&g.push({elem:i,handlers:d})}return h<b.length&&g.push({elem:this,handlers:b.slice(h)}),g},fix:function(a){if(a[n.expando])return a;var b,c,e,f=a.type,g=a,h=this.fixHooks[f];h||(this.fixHooks[f]=h=ma.test(f)?this.mouseHooks:la.test(f)?this.keyHooks:{}),e=h.props?this.props.concat(h.props):this.props,a=new n.Event(g),b=e.length;while(b--)c=e[b],a[c]=g[c];return a.target||(a.target=g.srcElement||d),3===a.target.nodeType&&(a.target=a.target.parentNode),a.metaKey=!!a.metaKey,h.filter?h.filter(a,g):a},props:"altKey bubbles cancelable ctrlKey currentTarget detail eventPhase metaKey relatedTarget shiftKey target timeStamp view which".split(" "),fixHooks:{},keyHooks:{props:"char charCode key keyCode".split(" "),filter:function(a,b){return null==a.which&&(a.which=null!=b.charCode?b.charCode:b.keyCode),a}},mouseHooks:{props:"button buttons clientX clientY fromElement offsetX offsetY pageX pageY screenX screenY toElement".split(" "),filter:function(a,b){var c,e,f,g=b.button,h=b.fromElement;return null==a.pageX&&null!=b.clientX&&(e=a.target.ownerDocument||d,f=e.documentElement,c=e.body,a.pageX=b.clientX+(f&&f.scrollLeft||c&&c.scrollLeft||0)-(f&&f.clientLeft||c&&c.clientLeft||0),a.pageY=b.clientY+(f&&f.scrollTop||c&&c.scrollTop||0)-(f&&f.clientTop||c&&c.clientTop||0)),!a.relatedTarget&&h&&(a.relatedTarget=h===a.target?b.toElement:h),a.which||void 0===g||(a.which=1&g?1:2&g?3:4&g?2:0),a}},special:{load:{noBubble:!0},focus:{trigger:function(){if(this!==ra()&&this.focus)try{return this.focus(),!1}catch(a){}},delegateType:"focusin"},blur:{trigger:function(){return this===ra()&&this.blur?(this.blur(),!1):void 0},delegateType:"focusout"},click:{trigger:function(){return n.nodeName(this,"input")&&"checkbox"===this.type&&this.click?(this.click(),!1):void 0},_default:function(a){return n.nodeName(a.target,"a")}},beforeunload:{postDispatch:function(a){void 0!==a.result&&a.originalEvent&&(a.originalEvent.returnValue=a.result)}}},simulate:function(a,b,c){var d=n.extend(new n.Event,c,{type:a,isSimulated:!0});n.event.trigger(d,null,b),d.isDefaultPrevented()&&c.preventDefault()}},n.removeEvent=d.removeEventListener?function(a,b,c){a.removeEventListener&&a.removeEventListener(b,c)}:function(a,b,c){var d="on"+b;a.detachEvent&&("undefined"==typeof a[d]&&(a[d]=null),a.detachEvent(d,c))},n.Event=function(a,b){return this instanceof n.Event?(a&&a.type?(this.originalEvent=a,this.type=a.type,this.isDefaultPrevented=a.defaultPrevented||void 0===a.defaultPrevented&&a.returnValue===!1?pa:qa):this.type=a,b&&n.extend(this,b),this.timeStamp=a&&a.timeStamp||n.now(),void(this[n.expando]=!0)):new n.Event(a,b)},n.Event.prototype={constructor:n.Event,isDefaultPrevented:qa,isPropagationStopped:qa,isImmediatePropagationStopped:qa,preventDefault:function(){var a=this.originalEvent;this.isDefaultPrevented=pa,a&&(a.preventDefault?a.preventDefault():a.returnValue=!1)},stopPropagation:function(){var a=this.originalEvent;this.isPropagationStopped=pa,a&&!this.isSimulated&&(a.stopPropagation&&a.stopPropagation(),a.cancelBubble=!0)},stopImmediatePropagation:function(){var a=this.originalEvent;this.isImmediatePropagationStopped=pa,a&&a.stopImmediatePropagation&&a.stopImmediatePropagation(),this.stopPropagation()}},n.each({mouseenter:"mouseover",mouseleave:"mouseout",pointerenter:"pointerover",pointerleave:"pointerout"},function(a,b){n.event.special[a]={delegateType:b,bindType:b,handle:function(a){var c,d=this,e=a.relatedTarget,f=a.handleObj;return e&&(e===d||n.contains(d,e))||(a.type=f.origType,c=f.handler.apply(this,arguments),a.type=b),c}}}),l.submit||(n.event.special.submit={setup:function(){return n.nodeName(this,"form")?!1:void n.event.add(this,"click._submit keypress._submit",function(a){var b=a.target,c=n.nodeName(b,"input")||n.nodeName(b,"button")?n.prop(b,"form"):void 0;c&&!n._data(c,"submit")&&(n.event.add(c,"submit._submit",function(a){a._submitBubble=!0}),n._data(c,"submit",!0))})},postDispatch:function(a){a._submitBubble&&(delete a._submitBubble,this.parentNode&&!a.isTrigger&&n.event.simulate("submit",this.parentNode,a))},teardown:function(){return n.nodeName(this,"form")?!1:void n.event.remove(this,"._submit")}}),l.change||(n.event.special.change={setup:function(){return ka.test(this.nodeName)?("checkbox"!==this.type&&"radio"!==this.type||(n.event.add(this,"propertychange._change",function(a){"checked"===a.originalEvent.propertyName&&(this._justChanged=!0)}),n.event.add(this,"click._change",function(a){this._justChanged&&!a.isTrigger&&(this._justChanged=!1),n.event.simulate("change",this,a)})),!1):void n.event.add(this,"beforeactivate._change",function(a){var b=a.target;ka.test(b.nodeName)&&!n._data(b,"change")&&(n.event.add(b,"change._change",function(a){!this.parentNode||a.isSimulated||a.isTrigger||n.event.simulate("change",this.parentNode,a)}),n._data(b,"change",!0))})},handle:function(a){var b=a.target;return this!==b||a.isSimulated||a.isTrigger||"radio"!==b.type&&"checkbox"!==b.type?a.handleObj.handler.apply(this,arguments):void 0},teardown:function(){return n.event.remove(this,"._change"),!ka.test(this.nodeName)}}),l.focusin||n.each({focus:"focusin",blur:"focusout"},function(a,b){var c=function(a){n.event.simulate(b,a.target,n.event.fix(a))};n.event.special[b]={setup:function(){var d=this.ownerDocument||this,e=n._data(d,b);e||d.addEventListener(a,c,!0),n._data(d,b,(e||0)+1)},teardown:function(){var d=this.ownerDocument||this,e=n._data(d,b)-1;e?n._data(d,b,e):(d.removeEventListener(a,c,!0),n._removeData(d,b))}}}),n.fn.extend({on:function(a,b,c,d){return sa(this,a,b,c,d)},one:function(a,b,c,d){return sa(this,a,b,c,d,1)},off:function(a,b,c){var d,e;if(a&&a.preventDefault&&a.handleObj)return d=a.handleObj,n(a.delegateTarget).off(d.namespace?d.origType+"."+d.namespace:d.origType,d.selector,d.handler),this;if("object"==typeof a){for(e in a)this.off(e,b,a[e]);return this}return b!==!1&&"function"!=typeof b||(c=b,b=void 0),c===!1&&(c=qa),this.each(function(){n.event.remove(this,a,c,b)})},trigger:function(a,b){return this.each(function(){n.event.trigger(a,b,this)})},triggerHandler:function(a,b){var c=this[0];return c?n.event.trigger(a,b,c,!0):void 0}});var ta=/ jQuery\d+="(?:null|\d+)"/g,ua=new RegExp("<(?:"+ba+")[\\s/>]","i"),va=/<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:-]+)[^>]*)\/>/gi,wa=/<script|<style|<link/i,xa=/checked\s*(?:[^=]|=\s*.checked.)/i,ya=/^true\/(.*)/,za=/^\s*<!(?:\[CDATA\[|--)|(?:\]\]|--)>\s*$/g,Aa=ca(d),Ba=Aa.appendChild(d.createElement("div"));function Ca(a,b){return n.nodeName(a,"table")&&n.nodeName(11!==b.nodeType?b:b.firstChild,"tr")?a.getElementsByTagName("tbody")[0]||a.appendChild(a.ownerDocument.createElement("tbody")):a}function Da(a){return a.type=(null!==n.find.attr(a,"type"))+"/"+a.type,a}function Ea(a){var b=ya.exec(a.type);return b?a.type=b[1]:a.removeAttribute("type"),a}function Fa(a,b){if(1===b.nodeType&&n.hasData(a)){var c,d,e,f=n._data(a),g=n._data(b,f),h=f.events;if(h){delete g.handle,g.events={};for(c in h)for(d=0,e=h[c].length;e>d;d++)n.event.add(b,c,h[c][d])}g.data&&(g.data=n.extend({},g.data))}}function Ga(a,b){var c,d,e;if(1===b.nodeType){if(c=b.nodeName.toLowerCase(),!l.noCloneEvent&&b[n.expando]){e=n._data(b);for(d in e.events)n.removeEvent(b,d,e.handle);b.removeAttribute(n.expando)}"script"===c&&b.text!==a.text?(Da(b).text=a.text,Ea(b)):"object"===c?(b.parentNode&&(b.outerHTML=a.outerHTML),l.html5Clone&&a.innerHTML&&!n.trim(b.innerHTML)&&(b.innerHTML=a.innerHTML)):"input"===c&&Z.test(a.type)?(b.defaultChecked=b.checked=a.checked,b.value!==a.value&&(b.value=a.value)):"option"===c?b.defaultSelected=b.selected=a.defaultSelected:"input"!==c&&"textarea"!==c||(b.defaultValue=a.defaultValue)}}function Ha(a,b,c,d){b=f.apply([],b);var e,g,h,i,j,k,m=0,o=a.length,p=o-1,q=b[0],r=n.isFunction(q);if(r||o>1&&"string"==typeof q&&!l.checkClone&&xa.test(q))return a.each(function(e){var f=a.eq(e);r&&(b[0]=q.call(this,e,f.html())),Ha(f,b,c,d)});if(o&&(k=ja(b,a[0].ownerDocument,!1,a,d),e=k.firstChild,1===k.childNodes.length&&(k=e),e||d)){for(i=n.map(ea(k,"script"),Da),h=i.length;o>m;m++)g=k,m!==p&&(g=n.clone(g,!0,!0),h&&n.merge(i,ea(g,"script"))),c.call(a[m],g,m);if(h)for(j=i[i.length-1].ownerDocument,n.map(i,Ea),m=0;h>m;m++)g=i[m],_.test(g.type||"")&&!n._data(g,"globalEval")&&n.contains(j,g)&&(g.src?n._evalUrl&&n._evalUrl(g.src):n.globalEval((g.text||g.textContent||g.innerHTML||"").replace(za,"")));k=e=null}return a}function Ia(a,b,c){for(var d,e=b?n.filter(b,a):a,f=0;null!=(d=e[f]);f++)c||1!==d.nodeType||n.cleanData(ea(d)),d.parentNode&&(c&&n.contains(d.ownerDocument,d)&&fa(ea(d,"script")),d.parentNode.removeChild(d));return a}n.extend({htmlPrefilter:function(a){return a.replace(va,"<$1></$2>")},clone:function(a,b,c){var d,e,f,g,h,i=n.contains(a.ownerDocument,a);if(l.html5Clone||n.isXMLDoc(a)||!ua.test("<"+a.nodeName+">")?f=a.cloneNode(!0):(Ba.innerHTML=a.outerHTML,Ba.removeChild(f=Ba.firstChild)),!(l.noCloneEvent&&l.noCloneChecked||1!==a.nodeType&&11!==a.nodeType||n.isXMLDoc(a)))for(d=ea(f),h=ea(a),g=0;null!=(e=h[g]);++g)d[g]&&Ga(e,d[g]);if(b)if(c)for(h=h||ea(a),d=d||ea(f),g=0;null!=(e=h[g]);g++)Fa(e,d[g]);else Fa(a,f);return d=ea(f,"script"),d.length>0&&fa(d,!i&&ea(a,"script")),d=h=e=null,f},cleanData:function(a,b){for(var d,e,f,g,h=0,i=n.expando,j=n.cache,k=l.attributes,m=n.event.special;null!=(d=a[h]);h++)if((b||M(d))&&(f=d[i],g=f&&j[f])){if(g.events)for(e in g.events)m[e]?n.event.remove(d,e):n.removeEvent(d,e,g.handle);j[f]&&(delete j[f],k||"undefined"==typeof d.removeAttribute?d[i]=void 0:d.removeAttribute(i),c.push(f))}}}),n.fn.extend({domManip:Ha,detach:function(a){return Ia(this,a,!0)},remove:function(a){return Ia(this,a)},text:function(a){return Y(this,function(a){return void 0===a?n.text(this):this.empty().append((this[0]&&this[0].ownerDocument||d).createTextNode(a))},null,a,arguments.length)},append:function(){return Ha(this,arguments,function(a){if(1===this.nodeType||11===this.nodeType||9===this.nodeType){var b=Ca(this,a);b.appendChild(a)}})},prepend:function(){return Ha(this,arguments,function(a){if(1===this.nodeType||11===this.nodeType||9===this.nodeType){var b=Ca(this,a);b.insertBefore(a,b.firstChild)}})},before:function(){return Ha(this,arguments,function(a){this.parentNode&&this.parentNode.insertBefore(a,this)})},after:function(){return Ha(this,arguments,function(a){this.parentNode&&this.parentNode.insertBefore(a,this.nextSibling)})},empty:function(){for(var a,b=0;null!=(a=this[b]);b++){1===a.nodeType&&n.cleanData(ea(a,!1));while(a.firstChild)a.removeChild(a.firstChild);a.options&&n.nodeName(a,"select")&&(a.options.length=0)}return this},clone:function(a,b){return a=null==a?!1:a,b=null==b?a:b,this.map(function(){return n.clone(this,a,b)})},html:function(a){return Y(this,function(a){var b=this[0]||{},c=0,d=this.length;if(void 0===a)return 1===b.nodeType?b.innerHTML.replace(ta,""):void 0;if("string"==typeof a&&!wa.test(a)&&(l.htmlSerialize||!ua.test(a))&&(l.leadingWhitespace||!aa.test(a))&&!da[($.exec(a)||["",""])[1].toLowerCase()]){a=n.htmlPrefilter(a);try{for(;d>c;c++)b=this[c]||{},1===b.nodeType&&(n.cleanData(ea(b,!1)),b.innerHTML=a);b=0}catch(e){}}b&&this.empty().append(a)},null,a,arguments.length)},replaceWith:function(){var a=[];return Ha(this,arguments,function(b){var c=this.parentNode;n.inArray(this,a)<0&&(n.cleanData(ea(this)),c&&c.replaceChild(b,this))},a)}}),n.each({appendTo:"append",prependTo:"prepend",insertBefore:"before",insertAfter:"after",replaceAll:"replaceWith"},function(a,b){n.fn[a]=function(a){for(var c,d=0,e=[],f=n(a),h=f.length-1;h>=d;d++)c=d===h?this:this.clone(!0),n(f[d])[b](c),g.apply(e,c.get());return this.pushStack(e)}});var Ja,Ka={HTML:"block",BODY:"block"};function La(a,b){var c=n(b.createElement(a)).appendTo(b.body),d=n.css(c[0],"display");return c.detach(),d}function Ma(a){var b=d,c=Ka[a];return c||(c=La(a,b),"none"!==c&&c||(Ja=(Ja||n("<iframe frameborder='0' width='0' height='0'/>")).appendTo(b.documentElement),b=(Ja[0].contentWindow||Ja[0].contentDocument).document,b.write(),b.close(),c=La(a,b),Ja.detach()),Ka[a]=c),c}var Na=/^margin/,Oa=new RegExp("^("+T+")(?!px)[a-z%]+$","i"),Pa=function(a,b,c,d){var e,f,g={};for(f in b)g[f]=a.style[f],a.style[f]=b[f];e=c.apply(a,d||[]);for(f in b)a.style[f]=g[f];return e},Qa=d.documentElement;!function(){var b,c,e,f,g,h,i=d.createElement("div"),j=d.createElement("div");if(j.style){j.style.cssText="float:left;opacity:.5",l.opacity="0.5"===j.style.opacity,l.cssFloat=!!j.style.cssFloat,j.style.backgroundClip="content-box",j.cloneNode(!0).style.backgroundClip="",l.clearCloneStyle="content-box"===j.style.backgroundClip,i=d.createElement("div"),i.style.cssText="border:0;width:8px;height:0;top:0;left:-9999px;padding:0;margin-top:1px;position:absolute",j.innerHTML="",i.appendChild(j),l.boxSizing=""===j.style.boxSizing||""===j.style.MozBoxSizing||""===j.style.WebkitBoxSizing,n.extend(l,{reliableHiddenOffsets:function(){return null==b&&k(),f},boxSizingReliable:function(){return null==b&&k(),e},pixelMarginRight:function(){return null==b&&k(),c},pixelPosition:function(){return null==b&&k(),b},reliableMarginRight:function(){return null==b&&k(),g},reliableMarginLeft:function(){return null==b&&k(),h}});function k(){var k,l,m=d.documentElement;m.appendChild(i),j.style.cssText="-webkit-box-sizing:border-box;box-sizing:border-box;position:relative;display:block;margin:auto;border:1px;padding:1px;top:1%;width:50%",b=e=h=!1,c=g=!0,a.getComputedStyle&&(l=a.getComputedStyle(j),b="1%"!==(l||{}).top,h="2px"===(l||{}).marginLeft,e="4px"===(l||{width:"4px"}).width,j.style.marginRight="50%",c="4px"===(l||{marginRight:"4px"}).marginRight,k=j.appendChild(d.createElement("div")),k.style.cssText=j.style.cssText="-webkit-box-sizing:content-box;-moz-box-sizing:content-box;box-sizing:content-box;display:block;margin:0;border:0;padding:0",k.style.marginRight=k.style.width="0",j.style.width="1px",g=!parseFloat((a.getComputedStyle(k)||{}).marginRight),j.removeChild(k)),j.style.display="none",f=0===j.getClientRects().length,f&&(j.style.display="",j.innerHTML="<table><tr><td></td><td>t</td></tr></table>",j.childNodes[0].style.borderCollapse="separate",k=j.getElementsByTagName("td"),k[0].style.cssText="margin:0;border:0;padding:0;display:none",f=0===k[0].offsetHeight,f&&(k[0].style.display="",k[1].style.display="none",f=0===k[0].offsetHeight)),m.removeChild(i)}}}();var Ra,Sa,Ta=/^(top|right|bottom|left)$/;a.getComputedStyle?(Ra=function(b){var c=b.ownerDocument.defaultView;return c&&c.opener||(c=a),c.getComputedStyle(b)},Sa=function(a,b,c){var d,e,f,g,h=a.style;return c=c||Ra(a),g=c?c.getPropertyValue(b)||c[b]:void 0,""!==g&&void 0!==g||n.contains(a.ownerDocument,a)||(g=n.style(a,b)),c&&!l.pixelMarginRight()&&Oa.test(g)&&Na.test(b)&&(d=h.width,e=h.minWidth,f=h.maxWidth,h.minWidth=h.maxWidth=h.width=g,g=c.width,h.width=d,h.minWidth=e,h.maxWidth=f),void 0===g?g:g+""}):Qa.currentStyle&&(Ra=function(a){return a.currentStyle},Sa=function(a,b,c){var d,e,f,g,h=a.style;return c=c||Ra(a),g=c?c[b]:void 0,null==g&&h&&h[b]&&(g=h[b]),Oa.test(g)&&!Ta.test(b)&&(d=h.left,e=a.runtimeStyle,f=e&&e.left,f&&(e.left=a.currentStyle.left),h.left="fontSize"===b?"1em":g,g=h.pixelLeft+"px",h.left=d,f&&(e.left=f)),void 0===g?g:g+""||"auto"});function Ua(a,b){return{get:function(){return a()?void delete this.get:(this.get=b).apply(this,arguments)}}}var Va=/alpha\([^)]*\)/i,Wa=/opacity\s*=\s*([^)]*)/i,Xa=/^(none|table(?!-c[ea]).+)/,Ya=new RegExp("^("+T+")(.*)$","i"),Za={position:"absolute",visibility:"hidden",display:"block"},$a={letterSpacing:"0",fontWeight:"400"},_a=["Webkit","O","Moz","ms"],ab=d.createElement("div").style;function bb(a){if(a in ab)return a;var b=a.charAt(0).toUpperCase()+a.slice(1),c=_a.length;while(c--)if(a=_a[c]+b,a in ab)return a}function cb(a,b){for(var c,d,e,f=[],g=0,h=a.length;h>g;g++)d=a[g],d.style&&(f[g]=n._data(d,"olddisplay"),c=d.style.display,b?(f[g]||"none"!==c||(d.style.display=""),""===d.style.display&&W(d)&&(f[g]=n._data(d,"olddisplay",Ma(d.nodeName)))):(e=W(d),(c&&"none"!==c||!e)&&n._data(d,"olddisplay",e?c:n.css(d,"display"))));for(g=0;h>g;g++)d=a[g],d.style&&(b&&"none"!==d.style.display&&""!==d.style.display||(d.style.display=b?f[g]||"":"none"));return a}function db(a,b,c){var d=Ya.exec(b);return d?Math.max(0,d[1]-(c||0))+(d[2]||"px"):b}function eb(a,b,c,d,e){for(var f=c===(d?"border":"content")?4:"width"===b?1:0,g=0;4>f;f+=2)"margin"===c&&(g+=n.css(a,c+V[f],!0,e)),d?("content"===c&&(g-=n.css(a,"padding"+V[f],!0,e)),"margin"!==c&&(g-=n.css(a,"border"+V[f]+"Width",!0,e))):(g+=n.css(a,"padding"+V[f],!0,e),"padding"!==c&&(g+=n.css(a,"border"+V[f]+"Width",!0,e)));return g}function fb(a,b,c){var d=!0,e="width"===b?a.offsetWidth:a.offsetHeight,f=Ra(a),g=l.boxSizing&&"border-box"===n.css(a,"boxSizing",!1,f);if(0>=e||null==e){if(e=Sa(a,b,f),(0>e||null==e)&&(e=a.style[b]),Oa.test(e))return e;d=g&&(l.boxSizingReliable()||e===a.style[b]),e=parseFloat(e)||0}return e+eb(a,b,c||(g?"border":"content"),d,f)+"px"}n.extend({cssHooks:{opacity:{get:function(a,b){if(b){var c=Sa(a,"opacity");return""===c?"1":c}}}},cssNumber:{animationIterationCount:!0,columnCount:!0,fillOpacity:!0,flexGrow:!0,flexShrink:!0,fontWeight:!0,lineHeight:!0,opacity:!0,order:!0,orphans:!0,widows:!0,zIndex:!0,zoom:!0},cssProps:{"float":l.cssFloat?"cssFloat":"styleFloat"},style:function(a,b,c,d){if(a&&3!==a.nodeType&&8!==a.nodeType&&a.style){var e,f,g,h=n.camelCase(b),i=a.style;if(b=n.cssProps[h]||(n.cssProps[h]=bb(h)||h),g=n.cssHooks[b]||n.cssHooks[h],void 0===c)return g&&"get"in g&&void 0!==(e=g.get(a,!1,d))?e:i[b];if(f=typeof c,"string"===f&&(e=U.exec(c))&&e[1]&&(c=X(a,b,e),f="number"),null!=c&&c===c&&("number"===f&&(c+=e&&e[3]||(n.cssNumber[h]?"":"px")),l.clearCloneStyle||""!==c||0!==b.indexOf("background")||(i[b]="inherit"),!(g&&"set"in g&&void 0===(c=g.set(a,c,d)))))try{i[b]=c}catch(j){}}},css:function(a,b,c,d){var e,f,g,h=n.camelCase(b);return b=n.cssProps[h]||(n.cssProps[h]=bb(h)||h),g=n.cssHooks[b]||n.cssHooks[h],g&&"get"in g&&(f=g.get(a,!0,c)),void 0===f&&(f=Sa(a,b,d)),"normal"===f&&b in $a&&(f=$a[b]),""===c||c?(e=parseFloat(f),c===!0||isFinite(e)?e||0:f):f}}),n.each(["height","width"],function(a,b){n.cssHooks[b]={get:function(a,c,d){return c?Xa.test(n.css(a,"display"))&&0===a.offsetWidth?Pa(a,Za,function(){return fb(a,b,d)}):fb(a,b,d):void 0},set:function(a,c,d){var e=d&&Ra(a);return db(a,c,d?eb(a,b,d,l.boxSizing&&"border-box"===n.css(a,"boxSizing",!1,e),e):0)}}}),l.opacity||(n.cssHooks.opacity={get:function(a,b){return Wa.test((b&&a.currentStyle?a.currentStyle.filter:a.style.filter)||"")?.01*parseFloat(RegExp.$1)+"":b?"1":""},set:function(a,b){var c=a.style,d=a.currentStyle,e=n.isNumeric(b)?"alpha(opacity="+100*b+")":"",f=d&&d.filter||c.filter||"";c.zoom=1,(b>=1||""===b)&&""===n.trim(f.replace(Va,""))&&c.removeAttribute&&(c.removeAttribute("filter"),""===b||d&&!d.filter)||(c.filter=Va.test(f)?f.replace(Va,e):f+" "+e)}}),n.cssHooks.marginRight=Ua(l.reliableMarginRight,function(a,b){return b?Pa(a,{display:"inline-block"},Sa,[a,"marginRight"]):void 0}),n.cssHooks.marginLeft=Ua(l.reliableMarginLeft,function(a,b){return b?(parseFloat(Sa(a,"marginLeft"))||(n.contains(a.ownerDocument,a)?a.getBoundingClientRect().left-Pa(a,{
marginLeft:0},function(){return a.getBoundingClientRect().left}):0))+"px":void 0}),n.each({margin:"",padding:"",border:"Width"},function(a,b){n.cssHooks[a+b]={expand:function(c){for(var d=0,e={},f="string"==typeof c?c.split(" "):[c];4>d;d++)e[a+V[d]+b]=f[d]||f[d-2]||f[0];return e}},Na.test(a)||(n.cssHooks[a+b].set=db)}),n.fn.extend({css:function(a,b){return Y(this,function(a,b,c){var d,e,f={},g=0;if(n.isArray(b)){for(d=Ra(a),e=b.length;e>g;g++)f[b[g]]=n.css(a,b[g],!1,d);return f}return void 0!==c?n.style(a,b,c):n.css(a,b)},a,b,arguments.length>1)},show:function(){return cb(this,!0)},hide:function(){return cb(this)},toggle:function(a){return"boolean"==typeof a?a?this.show():this.hide():this.each(function(){W(this)?n(this).show():n(this).hide()})}});function gb(a,b,c,d,e){return new gb.prototype.init(a,b,c,d,e)}n.Tween=gb,gb.prototype={constructor:gb,init:function(a,b,c,d,e,f){this.elem=a,this.prop=c,this.easing=e||n.easing._default,this.options=b,this.start=this.now=this.cur(),this.end=d,this.unit=f||(n.cssNumber[c]?"":"px")},cur:function(){var a=gb.propHooks[this.prop];return a&&a.get?a.get(this):gb.propHooks._default.get(this)},run:function(a){var b,c=gb.propHooks[this.prop];return this.options.duration?this.pos=b=n.easing[this.easing](a,this.options.duration*a,0,1,this.options.duration):this.pos=b=a,this.now=(this.end-this.start)*b+this.start,this.options.step&&this.options.step.call(this.elem,this.now,this),c&&c.set?c.set(this):gb.propHooks._default.set(this),this}},gb.prototype.init.prototype=gb.prototype,gb.propHooks={_default:{get:function(a){var b;return 1!==a.elem.nodeType||null!=a.elem[a.prop]&&null==a.elem.style[a.prop]?a.elem[a.prop]:(b=n.css(a.elem,a.prop,""),b&&"auto"!==b?b:0)},set:function(a){n.fx.step[a.prop]?n.fx.step[a.prop](a):1!==a.elem.nodeType||null==a.elem.style[n.cssProps[a.prop]]&&!n.cssHooks[a.prop]?a.elem[a.prop]=a.now:n.style(a.elem,a.prop,a.now+a.unit)}}},gb.propHooks.scrollTop=gb.propHooks.scrollLeft={set:function(a){a.elem.nodeType&&a.elem.parentNode&&(a.elem[a.prop]=a.now)}},n.easing={linear:function(a){return a},swing:function(a){return.5-Math.cos(a*Math.PI)/2},_default:"swing"},n.fx=gb.prototype.init,n.fx.step={};var hb,ib,jb=/^(?:toggle|show|hide)$/,kb=/queueHooks$/;function lb(){return a.setTimeout(function(){hb=void 0}),hb=n.now()}function mb(a,b){var c,d={height:a},e=0;for(b=b?1:0;4>e;e+=2-b)c=V[e],d["margin"+c]=d["padding"+c]=a;return b&&(d.opacity=d.width=a),d}function nb(a,b,c){for(var d,e=(qb.tweeners[b]||[]).concat(qb.tweeners["*"]),f=0,g=e.length;g>f;f++)if(d=e[f].call(c,b,a))return d}function ob(a,b,c){var d,e,f,g,h,i,j,k,m=this,o={},p=a.style,q=a.nodeType&&W(a),r=n._data(a,"fxshow");c.queue||(h=n._queueHooks(a,"fx"),null==h.unqueued&&(h.unqueued=0,i=h.empty.fire,h.empty.fire=function(){h.unqueued||i()}),h.unqueued++,m.always(function(){m.always(function(){h.unqueued--,n.queue(a,"fx").length||h.empty.fire()})})),1===a.nodeType&&("height"in b||"width"in b)&&(c.overflow=[p.overflow,p.overflowX,p.overflowY],j=n.css(a,"display"),k="none"===j?n._data(a,"olddisplay")||Ma(a.nodeName):j,"inline"===k&&"none"===n.css(a,"float")&&(l.inlineBlockNeedsLayout&&"inline"!==Ma(a.nodeName)?p.zoom=1:p.display="inline-block")),c.overflow&&(p.overflow="hidden",l.shrinkWrapBlocks()||m.always(function(){p.overflow=c.overflow[0],p.overflowX=c.overflow[1],p.overflowY=c.overflow[2]}));for(d in b)if(e=b[d],jb.exec(e)){if(delete b[d],f=f||"toggle"===e,e===(q?"hide":"show")){if("show"!==e||!r||void 0===r[d])continue;q=!0}o[d]=r&&r[d]||n.style(a,d)}else j=void 0;if(n.isEmptyObject(o))"inline"===("none"===j?Ma(a.nodeName):j)&&(p.display=j);else{r?"hidden"in r&&(q=r.hidden):r=n._data(a,"fxshow",{}),f&&(r.hidden=!q),q?n(a).show():m.done(function(){n(a).hide()}),m.done(function(){var b;n._removeData(a,"fxshow");for(b in o)n.style(a,b,o[b])});for(d in o)g=nb(q?r[d]:0,d,m),d in r||(r[d]=g.start,q&&(g.end=g.start,g.start="width"===d||"height"===d?1:0))}}function pb(a,b){var c,d,e,f,g;for(c in a)if(d=n.camelCase(c),e=b[d],f=a[c],n.isArray(f)&&(e=f[1],f=a[c]=f[0]),c!==d&&(a[d]=f,delete a[c]),g=n.cssHooks[d],g&&"expand"in g){f=g.expand(f),delete a[d];for(c in f)c in a||(a[c]=f[c],b[c]=e)}else b[d]=e}function qb(a,b,c){var d,e,f=0,g=qb.prefilters.length,h=n.Deferred().always(function(){delete i.elem}),i=function(){if(e)return!1;for(var b=hb||lb(),c=Math.max(0,j.startTime+j.duration-b),d=c/j.duration||0,f=1-d,g=0,i=j.tweens.length;i>g;g++)j.tweens[g].run(f);return h.notifyWith(a,[j,f,c]),1>f&&i?c:(h.resolveWith(a,[j]),!1)},j=h.promise({elem:a,props:n.extend({},b),opts:n.extend(!0,{specialEasing:{},easing:n.easing._default},c),originalProperties:b,originalOptions:c,startTime:hb||lb(),duration:c.duration,tweens:[],createTween:function(b,c){var d=n.Tween(a,j.opts,b,c,j.opts.specialEasing[b]||j.opts.easing);return j.tweens.push(d),d},stop:function(b){var c=0,d=b?j.tweens.length:0;if(e)return this;for(e=!0;d>c;c++)j.tweens[c].run(1);return b?(h.notifyWith(a,[j,1,0]),h.resolveWith(a,[j,b])):h.rejectWith(a,[j,b]),this}}),k=j.props;for(pb(k,j.opts.specialEasing);g>f;f++)if(d=qb.prefilters[f].call(j,a,k,j.opts))return n.isFunction(d.stop)&&(n._queueHooks(j.elem,j.opts.queue).stop=n.proxy(d.stop,d)),d;return n.map(k,nb,j),n.isFunction(j.opts.start)&&j.opts.start.call(a,j),n.fx.timer(n.extend(i,{elem:a,anim:j,queue:j.opts.queue})),j.progress(j.opts.progress).done(j.opts.done,j.opts.complete).fail(j.opts.fail).always(j.opts.always)}n.Animation=n.extend(qb,{tweeners:{"*":[function(a,b){var c=this.createTween(a,b);return X(c.elem,a,U.exec(b),c),c}]},tweener:function(a,b){n.isFunction(a)?(b=a,a=["*"]):a=a.match(G);for(var c,d=0,e=a.length;e>d;d++)c=a[d],qb.tweeners[c]=qb.tweeners[c]||[],qb.tweeners[c].unshift(b)},prefilters:[ob],prefilter:function(a,b){b?qb.prefilters.unshift(a):qb.prefilters.push(a)}}),n.speed=function(a,b,c){var d=a&&"object"==typeof a?n.extend({},a):{complete:c||!c&&b||n.isFunction(a)&&a,duration:a,easing:c&&b||b&&!n.isFunction(b)&&b};return d.duration=n.fx.off?0:"number"==typeof d.duration?d.duration:d.duration in n.fx.speeds?n.fx.speeds[d.duration]:n.fx.speeds._default,null!=d.queue&&d.queue!==!0||(d.queue="fx"),d.old=d.complete,d.complete=function(){n.isFunction(d.old)&&d.old.call(this),d.queue&&n.dequeue(this,d.queue)},d},n.fn.extend({fadeTo:function(a,b,c,d){return this.filter(W).css("opacity",0).show().end().animate({opacity:b},a,c,d)},animate:function(a,b,c,d){var e=n.isEmptyObject(a),f=n.speed(b,c,d),g=function(){var b=qb(this,n.extend({},a),f);(e||n._data(this,"finish"))&&b.stop(!0)};return g.finish=g,e||f.queue===!1?this.each(g):this.queue(f.queue,g)},stop:function(a,b,c){var d=function(a){var b=a.stop;delete a.stop,b(c)};return"string"!=typeof a&&(c=b,b=a,a=void 0),b&&a!==!1&&this.queue(a||"fx",[]),this.each(function(){var b=!0,e=null!=a&&a+"queueHooks",f=n.timers,g=n._data(this);if(e)g[e]&&g[e].stop&&d(g[e]);else for(e in g)g[e]&&g[e].stop&&kb.test(e)&&d(g[e]);for(e=f.length;e--;)f[e].elem!==this||null!=a&&f[e].queue!==a||(f[e].anim.stop(c),b=!1,f.splice(e,1));!b&&c||n.dequeue(this,a)})},finish:function(a){return a!==!1&&(a=a||"fx"),this.each(function(){var b,c=n._data(this),d=c[a+"queue"],e=c[a+"queueHooks"],f=n.timers,g=d?d.length:0;for(c.finish=!0,n.queue(this,a,[]),e&&e.stop&&e.stop.call(this,!0),b=f.length;b--;)f[b].elem===this&&f[b].queue===a&&(f[b].anim.stop(!0),f.splice(b,1));for(b=0;g>b;b++)d[b]&&d[b].finish&&d[b].finish.call(this);delete c.finish})}}),n.each(["toggle","show","hide"],function(a,b){var c=n.fn[b];n.fn[b]=function(a,d,e){return null==a||"boolean"==typeof a?c.apply(this,arguments):this.animate(mb(b,!0),a,d,e)}}),n.each({slideDown:mb("show"),slideUp:mb("hide"),slideToggle:mb("toggle"),fadeIn:{opacity:"show"},fadeOut:{opacity:"hide"},fadeToggle:{opacity:"toggle"}},function(a,b){n.fn[a]=function(a,c,d){return this.animate(b,a,c,d)}}),n.timers=[],n.fx.tick=function(){var a,b=n.timers,c=0;for(hb=n.now();c<b.length;c++)a=b[c],a()||b[c]!==a||b.splice(c--,1);b.length||n.fx.stop(),hb=void 0},n.fx.timer=function(a){n.timers.push(a),a()?n.fx.start():n.timers.pop()},n.fx.interval=13,n.fx.start=function(){ib||(ib=a.setInterval(n.fx.tick,n.fx.interval))},n.fx.stop=function(){a.clearInterval(ib),ib=null},n.fx.speeds={slow:600,fast:200,_default:400},n.fn.delay=function(b,c){return b=n.fx?n.fx.speeds[b]||b:b,c=c||"fx",this.queue(c,function(c,d){var e=a.setTimeout(c,b);d.stop=function(){a.clearTimeout(e)}})},function(){var a,b=d.createElement("input"),c=d.createElement("div"),e=d.createElement("select"),f=e.appendChild(d.createElement("option"));c=d.createElement("div"),c.setAttribute("className","t"),c.innerHTML="  <link/><table></table><a href='/a'>a</a><input type='checkbox'/>",a=c.getElementsByTagName("a")[0],b.setAttribute("type","checkbox"),c.appendChild(b),a=c.getElementsByTagName("a")[0],a.style.cssText="top:1px",l.getSetAttribute="t"!==c.className,l.style=/top/.test(a.getAttribute("style")),l.hrefNormalized="/a"===a.getAttribute("href"),l.checkOn=!!b.value,l.optSelected=f.selected,l.enctype=!!d.createElement("form").enctype,e.disabled=!0,l.optDisabled=!f.disabled,b=d.createElement("input"),b.setAttribute("value",""),l.input=""===b.getAttribute("value"),b.value="t",b.setAttribute("type","radio"),l.radioValue="t"===b.value}();var rb=/\r/g,sb=/[\x20\t\r\n\f]+/g;n.fn.extend({val:function(a){var b,c,d,e=this[0];{if(arguments.length)return d=n.isFunction(a),this.each(function(c){var e;1===this.nodeType&&(e=d?a.call(this,c,n(this).val()):a,null==e?e="":"number"==typeof e?e+="":n.isArray(e)&&(e=n.map(e,function(a){return null==a?"":a+""})),b=n.valHooks[this.type]||n.valHooks[this.nodeName.toLowerCase()],b&&"set"in b&&void 0!==b.set(this,e,"value")||(this.value=e))});if(e)return b=n.valHooks[e.type]||n.valHooks[e.nodeName.toLowerCase()],b&&"get"in b&&void 0!==(c=b.get(e,"value"))?c:(c=e.value,"string"==typeof c?c.replace(rb,""):null==c?"":c)}}}),n.extend({valHooks:{option:{get:function(a){var b=n.find.attr(a,"value");return null!=b?b:n.trim(n.text(a)).replace(sb," ")}},select:{get:function(a){for(var b,c,d=a.options,e=a.selectedIndex,f="select-one"===a.type||0>e,g=f?null:[],h=f?e+1:d.length,i=0>e?h:f?e:0;h>i;i++)if(c=d[i],(c.selected||i===e)&&(l.optDisabled?!c.disabled:null===c.getAttribute("disabled"))&&(!c.parentNode.disabled||!n.nodeName(c.parentNode,"optgroup"))){if(b=n(c).val(),f)return b;g.push(b)}return g},set:function(a,b){var c,d,e=a.options,f=n.makeArray(b),g=e.length;while(g--)if(d=e[g],n.inArray(n.valHooks.option.get(d),f)>-1)try{d.selected=c=!0}catch(h){d.scrollHeight}else d.selected=!1;return c||(a.selectedIndex=-1),e}}}}),n.each(["radio","checkbox"],function(){n.valHooks[this]={set:function(a,b){return n.isArray(b)?a.checked=n.inArray(n(a).val(),b)>-1:void 0}},l.checkOn||(n.valHooks[this].get=function(a){return null===a.getAttribute("value")?"on":a.value})});var tb,ub,vb=n.expr.attrHandle,wb=/^(?:checked|selected)$/i,xb=l.getSetAttribute,yb=l.input;n.fn.extend({attr:function(a,b){return Y(this,n.attr,a,b,arguments.length>1)},removeAttr:function(a){return this.each(function(){n.removeAttr(this,a)})}}),n.extend({attr:function(a,b,c){var d,e,f=a.nodeType;if(3!==f&&8!==f&&2!==f)return"undefined"==typeof a.getAttribute?n.prop(a,b,c):(1===f&&n.isXMLDoc(a)||(b=b.toLowerCase(),e=n.attrHooks[b]||(n.expr.match.bool.test(b)?ub:tb)),void 0!==c?null===c?void n.removeAttr(a,b):e&&"set"in e&&void 0!==(d=e.set(a,c,b))?d:(a.setAttribute(b,c+""),c):e&&"get"in e&&null!==(d=e.get(a,b))?d:(d=n.find.attr(a,b),null==d?void 0:d))},attrHooks:{type:{set:function(a,b){if(!l.radioValue&&"radio"===b&&n.nodeName(a,"input")){var c=a.value;return a.setAttribute("type",b),c&&(a.value=c),b}}}},removeAttr:function(a,b){var c,d,e=0,f=b&&b.match(G);if(f&&1===a.nodeType)while(c=f[e++])d=n.propFix[c]||c,n.expr.match.bool.test(c)?yb&&xb||!wb.test(c)?a[d]=!1:a[n.camelCase("default-"+c)]=a[d]=!1:n.attr(a,c,""),a.removeAttribute(xb?c:d)}}),ub={set:function(a,b,c){return b===!1?n.removeAttr(a,c):yb&&xb||!wb.test(c)?a.setAttribute(!xb&&n.propFix[c]||c,c):a[n.camelCase("default-"+c)]=a[c]=!0,c}},n.each(n.expr.match.bool.source.match(/\w+/g),function(a,b){var c=vb[b]||n.find.attr;yb&&xb||!wb.test(b)?vb[b]=function(a,b,d){var e,f;return d||(f=vb[b],vb[b]=e,e=null!=c(a,b,d)?b.toLowerCase():null,vb[b]=f),e}:vb[b]=function(a,b,c){return c?void 0:a[n.camelCase("default-"+b)]?b.toLowerCase():null}}),yb&&xb||(n.attrHooks.value={set:function(a,b,c){return n.nodeName(a,"input")?void(a.defaultValue=b):tb&&tb.set(a,b,c)}}),xb||(tb={set:function(a,b,c){var d=a.getAttributeNode(c);return d||a.setAttributeNode(d=a.ownerDocument.createAttribute(c)),d.value=b+="","value"===c||b===a.getAttribute(c)?b:void 0}},vb.id=vb.name=vb.coords=function(a,b,c){var d;return c?void 0:(d=a.getAttributeNode(b))&&""!==d.value?d.value:null},n.valHooks.button={get:function(a,b){var c=a.getAttributeNode(b);return c&&c.specified?c.value:void 0},set:tb.set},n.attrHooks.contenteditable={set:function(a,b,c){tb.set(a,""===b?!1:b,c)}},n.each(["width","height"],function(a,b){n.attrHooks[b]={set:function(a,c){return""===c?(a.setAttribute(b,"auto"),c):void 0}}})),l.style||(n.attrHooks.style={get:function(a){return a.style.cssText||void 0},set:function(a,b){return a.style.cssText=b+""}});var zb=/^(?:input|select|textarea|button|object)$/i,Ab=/^(?:a|area)$/i;n.fn.extend({prop:function(a,b){return Y(this,n.prop,a,b,arguments.length>1)},removeProp:function(a){return a=n.propFix[a]||a,this.each(function(){try{this[a]=void 0,delete this[a]}catch(b){}})}}),n.extend({prop:function(a,b,c){var d,e,f=a.nodeType;if(3!==f&&8!==f&&2!==f)return 1===f&&n.isXMLDoc(a)||(b=n.propFix[b]||b,e=n.propHooks[b]),void 0!==c?e&&"set"in e&&void 0!==(d=e.set(a,c,b))?d:a[b]=c:e&&"get"in e&&null!==(d=e.get(a,b))?d:a[b]},propHooks:{tabIndex:{get:function(a){var b=n.find.attr(a,"tabindex");return b?parseInt(b,10):zb.test(a.nodeName)||Ab.test(a.nodeName)&&a.href?0:-1}}},propFix:{"for":"htmlFor","class":"className"}}),l.hrefNormalized||n.each(["href","src"],function(a,b){n.propHooks[b]={get:function(a){return a.getAttribute(b,4)}}}),l.optSelected||(n.propHooks.selected={get:function(a){var b=a.parentNode;return b&&(b.selectedIndex,b.parentNode&&b.parentNode.selectedIndex),null},set:function(a){var b=a.parentNode;b&&(b.selectedIndex,b.parentNode&&b.parentNode.selectedIndex)}}),n.each(["tabIndex","readOnly","maxLength","cellSpacing","cellPadding","rowSpan","colSpan","useMap","frameBorder","contentEditable"],function(){n.propFix[this.toLowerCase()]=this}),l.enctype||(n.propFix.enctype="encoding");var Bb=/[\t\r\n\f]/g;function Cb(a){return n.attr(a,"class")||""}n.fn.extend({addClass:function(a){var b,c,d,e,f,g,h,i=0;if(n.isFunction(a))return this.each(function(b){n(this).addClass(a.call(this,b,Cb(this)))});if("string"==typeof a&&a){b=a.match(G)||[];while(c=this[i++])if(e=Cb(c),d=1===c.nodeType&&(" "+e+" ").replace(Bb," ")){g=0;while(f=b[g++])d.indexOf(" "+f+" ")<0&&(d+=f+" ");h=n.trim(d),e!==h&&n.attr(c,"class",h)}}return this},removeClass:function(a){var b,c,d,e,f,g,h,i=0;if(n.isFunction(a))return this.each(function(b){n(this).removeClass(a.call(this,b,Cb(this)))});if(!arguments.length)return this.attr("class","");if("string"==typeof a&&a){b=a.match(G)||[];while(c=this[i++])if(e=Cb(c),d=1===c.nodeType&&(" "+e+" ").replace(Bb," ")){g=0;while(f=b[g++])while(d.indexOf(" "+f+" ")>-1)d=d.replace(" "+f+" "," ");h=n.trim(d),e!==h&&n.attr(c,"class",h)}}return this},toggleClass:function(a,b){var c=typeof a;return"boolean"==typeof b&&"string"===c?b?this.addClass(a):this.removeClass(a):n.isFunction(a)?this.each(function(c){n(this).toggleClass(a.call(this,c,Cb(this),b),b)}):this.each(function(){var b,d,e,f;if("string"===c){d=0,e=n(this),f=a.match(G)||[];while(b=f[d++])e.hasClass(b)?e.removeClass(b):e.addClass(b)}else void 0!==a&&"boolean"!==c||(b=Cb(this),b&&n._data(this,"__className__",b),n.attr(this,"class",b||a===!1?"":n._data(this,"__className__")||""))})},hasClass:function(a){var b,c,d=0;b=" "+a+" ";while(c=this[d++])if(1===c.nodeType&&(" "+Cb(c)+" ").replace(Bb," ").indexOf(b)>-1)return!0;return!1}}),n.each("blur focus focusin focusout load resize scroll unload click dblclick mousedown mouseup mousemove mouseover mouseout mouseenter mouseleave change select submit keydown keypress keyup error contextmenu".split(" "),function(a,b){n.fn[b]=function(a,c){return arguments.length>0?this.on(b,null,a,c):this.trigger(b)}}),n.fn.extend({hover:function(a,b){return this.mouseenter(a).mouseleave(b||a)}});var Db=a.location,Eb=n.now(),Fb=/\?/,Gb=/(,)|(\[|{)|(}|])|"(?:[^"\\\r\n]|\\["\\\/bfnrt]|\\u[\da-fA-F]{4})*"\s*:?|true|false|null|-?(?!0\d)\d+(?:\.\d+|)(?:[eE][+-]?\d+|)/g;n.parseJSON=function(b){if(a.JSON&&a.JSON.parse)return a.JSON.parse(b+"");var c,d=null,e=n.trim(b+"");return e&&!n.trim(e.replace(Gb,function(a,b,e,f){return c&&b&&(d=0),0===d?a:(c=e||b,d+=!f-!e,"")}))?Function("return "+e)():n.error("Invalid JSON: "+b)},n.parseXML=function(b){var c,d;if(!b||"string"!=typeof b)return null;try{a.DOMParser?(d=new a.DOMParser,c=d.parseFromString(b,"text/xml")):(c=new a.ActiveXObject("Microsoft.XMLDOM"),c.async="false",c.loadXML(b))}catch(e){c=void 0}return c&&c.documentElement&&!c.getElementsByTagName("parsererror").length||n.error("Invalid XML: "+b),c};var Hb=/#.*$/,Ib=/([?&])_=[^&]*/,Jb=/^(.*?):[ \t]*([^\r\n]*)\r?$/gm,Kb=/^(?:about|app|app-storage|.+-extension|file|res|widget):$/,Lb=/^(?:GET|HEAD)$/,Mb=/^\/\//,Nb=/^([\w.+-]+:)(?:\/\/(?:[^\/?#]*@|)([^\/?#:]*)(?::(\d+)|)|)/,Ob={},Pb={},Qb="*/".concat("*"),Rb=Db.href,Sb=Nb.exec(Rb.toLowerCase())||[];function Tb(a){return function(b,c){"string"!=typeof b&&(c=b,b="*");var d,e=0,f=b.toLowerCase().match(G)||[];if(n.isFunction(c))while(d=f[e++])"+"===d.charAt(0)?(d=d.slice(1)||"*",(a[d]=a[d]||[]).unshift(c)):(a[d]=a[d]||[]).push(c)}}function Ub(a,b,c,d){var e={},f=a===Pb;function g(h){var i;return e[h]=!0,n.each(a[h]||[],function(a,h){var j=h(b,c,d);return"string"!=typeof j||f||e[j]?f?!(i=j):void 0:(b.dataTypes.unshift(j),g(j),!1)}),i}return g(b.dataTypes[0])||!e["*"]&&g("*")}function Vb(a,b){var c,d,e=n.ajaxSettings.flatOptions||{};for(d in b)void 0!==b[d]&&((e[d]?a:c||(c={}))[d]=b[d]);return c&&n.extend(!0,a,c),a}function Wb(a,b,c){var d,e,f,g,h=a.contents,i=a.dataTypes;while("*"===i[0])i.shift(),void 0===e&&(e=a.mimeType||b.getResponseHeader("Content-Type"));if(e)for(g in h)if(h[g]&&h[g].test(e)){i.unshift(g);break}if(i[0]in c)f=i[0];else{for(g in c){if(!i[0]||a.converters[g+" "+i[0]]){f=g;break}d||(d=g)}f=f||d}return f?(f!==i[0]&&i.unshift(f),c[f]):void 0}function Xb(a,b,c,d){var e,f,g,h,i,j={},k=a.dataTypes.slice();if(k[1])for(g in a.converters)j[g.toLowerCase()]=a.converters[g];f=k.shift();while(f)if(a.responseFields[f]&&(c[a.responseFields[f]]=b),!i&&d&&a.dataFilter&&(b=a.dataFilter(b,a.dataType)),i=f,f=k.shift())if("*"===f)f=i;else if("*"!==i&&i!==f){if(g=j[i+" "+f]||j["* "+f],!g)for(e in j)if(h=e.split(" "),h[1]===f&&(g=j[i+" "+h[0]]||j["* "+h[0]])){g===!0?g=j[e]:j[e]!==!0&&(f=h[0],k.unshift(h[1]));break}if(g!==!0)if(g&&a["throws"])b=g(b);else try{b=g(b)}catch(l){return{state:"parsererror",error:g?l:"No conversion from "+i+" to "+f}}}return{state:"success",data:b}}n.extend({active:0,lastModified:{},etag:{},ajaxSettings:{url:Rb,type:"GET",isLocal:Kb.test(Sb[1]),global:!0,processData:!0,async:!0,contentType:"application/x-www-form-urlencoded; charset=UTF-8",accepts:{"*":Qb,text:"text/plain",html:"text/html",xml:"application/xml, text/xml",json:"application/json, text/javascript"},contents:{xml:/\bxml\b/,html:/\bhtml/,json:/\bjson\b/},responseFields:{xml:"responseXML",text:"responseText",json:"responseJSON"},converters:{"* text":String,"text html":!0,"text json":n.parseJSON,"text xml":n.parseXML},flatOptions:{url:!0,context:!0}},ajaxSetup:function(a,b){return b?Vb(Vb(a,n.ajaxSettings),b):Vb(n.ajaxSettings,a)},ajaxPrefilter:Tb(Ob),ajaxTransport:Tb(Pb),ajax:function(b,c){"object"==typeof b&&(c=b,b=void 0),c=c||{};var d,e,f,g,h,i,j,k,l=n.ajaxSetup({},c),m=l.context||l,o=l.context&&(m.nodeType||m.jquery)?n(m):n.event,p=n.Deferred(),q=n.Callbacks("once memory"),r=l.statusCode||{},s={},t={},u=0,v="canceled",w={readyState:0,getResponseHeader:function(a){var b;if(2===u){if(!k){k={};while(b=Jb.exec(g))k[b[1].toLowerCase()]=b[2]}b=k[a.toLowerCase()]}return null==b?null:b},getAllResponseHeaders:function(){return 2===u?g:null},setRequestHeader:function(a,b){var c=a.toLowerCase();return u||(a=t[c]=t[c]||a,s[a]=b),this},overrideMimeType:function(a){return u||(l.mimeType=a),this},statusCode:function(a){var b;if(a)if(2>u)for(b in a)r[b]=[r[b],a[b]];else w.always(a[w.status]);return this},abort:function(a){var b=a||v;return j&&j.abort(b),y(0,b),this}};if(p.promise(w).complete=q.add,w.success=w.done,w.error=w.fail,l.url=((b||l.url||Rb)+"").replace(Hb,"").replace(Mb,Sb[1]+"//"),l.type=c.method||c.type||l.method||l.type,l.dataTypes=n.trim(l.dataType||"*").toLowerCase().match(G)||[""],null==l.crossDomain&&(d=Nb.exec(l.url.toLowerCase()),l.crossDomain=!(!d||d[1]===Sb[1]&&d[2]===Sb[2]&&(d[3]||("http:"===d[1]?"80":"443"))===(Sb[3]||("http:"===Sb[1]?"80":"443")))),l.data&&l.processData&&"string"!=typeof l.data&&(l.data=n.param(l.data,l.traditional)),Ub(Ob,l,c,w),2===u)return w;i=n.event&&l.global,i&&0===n.active++&&n.event.trigger("ajaxStart"),l.type=l.type.toUpperCase(),l.hasContent=!Lb.test(l.type),f=l.url,l.hasContent||(l.data&&(f=l.url+=(Fb.test(f)?"&":"?")+l.data,delete l.data),l.cache===!1&&(l.url=Ib.test(f)?f.replace(Ib,"$1_="+Eb++):f+(Fb.test(f)?"&":"?")+"_="+Eb++)),l.ifModified&&(n.lastModified[f]&&w.setRequestHeader("If-Modified-Since",n.lastModified[f]),n.etag[f]&&w.setRequestHeader("If-None-Match",n.etag[f])),(l.data&&l.hasContent&&l.contentType!==!1||c.contentType)&&w.setRequestHeader("Content-Type",l.contentType),w.setRequestHeader("Accept",l.dataTypes[0]&&l.accepts[l.dataTypes[0]]?l.accepts[l.dataTypes[0]]+("*"!==l.dataTypes[0]?", "+Qb+"; q=0.01":""):l.accepts["*"]);for(e in l.headers)w.setRequestHeader(e,l.headers[e]);if(l.beforeSend&&(l.beforeSend.call(m,w,l)===!1||2===u))return w.abort();v="abort";for(e in{success:1,error:1,complete:1})w[e](l[e]);if(j=Ub(Pb,l,c,w)){if(w.readyState=1,i&&o.trigger("ajaxSend",[w,l]),2===u)return w;l.async&&l.timeout>0&&(h=a.setTimeout(function(){w.abort("timeout")},l.timeout));try{u=1,j.send(s,y)}catch(x){if(!(2>u))throw x;y(-1,x)}}else y(-1,"No Transport");function y(b,c,d,e){var k,s,t,v,x,y=c;2!==u&&(u=2,h&&a.clearTimeout(h),j=void 0,g=e||"",w.readyState=b>0?4:0,k=b>=200&&300>b||304===b,d&&(v=Wb(l,w,d)),v=Xb(l,v,w,k),k?(l.ifModified&&(x=w.getResponseHeader("Last-Modified"),x&&(n.lastModified[f]=x),x=w.getResponseHeader("etag"),x&&(n.etag[f]=x)),204===b||"HEAD"===l.type?y="nocontent":304===b?y="notmodified":(y=v.state,s=v.data,t=v.error,k=!t)):(t=y,!b&&y||(y="error",0>b&&(b=0))),w.status=b,w.statusText=(c||y)+"",k?p.resolveWith(m,[s,y,w]):p.rejectWith(m,[w,y,t]),w.statusCode(r),r=void 0,i&&o.trigger(k?"ajaxSuccess":"ajaxError",[w,l,k?s:t]),q.fireWith(m,[w,y]),i&&(o.trigger("ajaxComplete",[w,l]),--n.active||n.event.trigger("ajaxStop")))}return w},getJSON:function(a,b,c){return n.get(a,b,c,"json")},getScript:function(a,b){return n.get(a,void 0,b,"script")}}),n.each(["get","post"],function(a,b){n[b]=function(a,c,d,e){return n.isFunction(c)&&(e=e||d,d=c,c=void 0),n.ajax(n.extend({url:a,type:b,dataType:e,data:c,success:d},n.isPlainObject(a)&&a))}}),n._evalUrl=function(a){return n.ajax({url:a,type:"GET",dataType:"script",cache:!0,async:!1,global:!1,"throws":!0})},n.fn.extend({wrapAll:function(a){if(n.isFunction(a))return this.each(function(b){n(this).wrapAll(a.call(this,b))});if(this[0]){var b=n(a,this[0].ownerDocument).eq(0).clone(!0);this[0].parentNode&&b.insertBefore(this[0]),b.map(function(){var a=this;while(a.firstChild&&1===a.firstChild.nodeType)a=a.firstChild;return a}).append(this)}return this},wrapInner:function(a){return n.isFunction(a)?this.each(function(b){n(this).wrapInner(a.call(this,b))}):this.each(function(){var b=n(this),c=b.contents();c.length?c.wrapAll(a):b.append(a)})},wrap:function(a){var b=n.isFunction(a);return this.each(function(c){n(this).wrapAll(b?a.call(this,c):a)})},unwrap:function(){return this.parent().each(function(){n.nodeName(this,"body")||n(this).replaceWith(this.childNodes)}).end()}});function Yb(a){return a.style&&a.style.display||n.css(a,"display")}function Zb(a){if(!n.contains(a.ownerDocument||d,a))return!0;while(a&&1===a.nodeType){if("none"===Yb(a)||"hidden"===a.type)return!0;a=a.parentNode}return!1}n.expr.filters.hidden=function(a){return l.reliableHiddenOffsets()?a.offsetWidth<=0&&a.offsetHeight<=0&&!a.getClientRects().length:Zb(a)},n.expr.filters.visible=function(a){return!n.expr.filters.hidden(a)};var $b=/%20/g,_b=/\[\]$/,ac=/\r?\n/g,bc=/^(?:submit|button|image|reset|file)$/i,cc=/^(?:input|select|textarea|keygen)/i;function dc(a,b,c,d){var e;if(n.isArray(b))n.each(b,function(b,e){c||_b.test(a)?d(a,e):dc(a+"["+("object"==typeof e&&null!=e?b:"")+"]",e,c,d)});else if(c||"object"!==n.type(b))d(a,b);else for(e in b)dc(a+"["+e+"]",b[e],c,d)}n.param=function(a,b){var c,d=[],e=function(a,b){b=n.isFunction(b)?b():null==b?"":b,d[d.length]=encodeURIComponent(a)+"="+encodeURIComponent(b)};if(void 0===b&&(b=n.ajaxSettings&&n.ajaxSettings.traditional),n.isArray(a)||a.jquery&&!n.isPlainObject(a))n.each(a,function(){e(this.name,this.value)});else for(c in a)dc(c,a[c],b,e);return d.join("&").replace($b,"+")},n.fn.extend({serialize:function(){return n.param(this.serializeArray())},serializeArray:function(){return this.map(function(){var a=n.prop(this,"elements");return a?n.makeArray(a):this}).filter(function(){var a=this.type;return this.name&&!n(this).is(":disabled")&&cc.test(this.nodeName)&&!bc.test(a)&&(this.checked||!Z.test(a))}).map(function(a,b){var c=n(this).val();return null==c?null:n.isArray(c)?n.map(c,function(a){return{name:b.name,value:a.replace(ac,"\r\n")}}):{name:b.name,value:c.replace(ac,"\r\n")}}).get()}}),n.ajaxSettings.xhr=void 0!==a.ActiveXObject?function(){return this.isLocal?ic():d.documentMode>8?hc():/^(get|post|head|put|delete|options)$/i.test(this.type)&&hc()||ic()}:hc;var ec=0,fc={},gc=n.ajaxSettings.xhr();a.attachEvent&&a.attachEvent("onunload",function(){for(var a in fc)fc[a](void 0,!0)}),l.cors=!!gc&&"withCredentials"in gc,gc=l.ajax=!!gc,gc&&n.ajaxTransport(function(b){if(!b.crossDomain||l.cors){var c;return{send:function(d,e){var f,g=b.xhr(),h=++ec;if(g.open(b.type,b.url,b.async,b.username,b.password),b.xhrFields)for(f in b.xhrFields)g[f]=b.xhrFields[f];b.mimeType&&g.overrideMimeType&&g.overrideMimeType(b.mimeType),b.crossDomain||d["X-Requested-With"]||(d["X-Requested-With"]="XMLHttpRequest");for(f in d)void 0!==d[f]&&g.setRequestHeader(f,d[f]+"");g.send(b.hasContent&&b.data||null),c=function(a,d){var f,i,j;if(c&&(d||4===g.readyState))if(delete fc[h],c=void 0,g.onreadystatechange=n.noop,d)4!==g.readyState&&g.abort();else{j={},f=g.status,"string"==typeof g.responseText&&(j.text=g.responseText);try{i=g.statusText}catch(k){i=""}f||!b.isLocal||b.crossDomain?1223===f&&(f=204):f=j.text?200:404}j&&e(f,i,j,g.getAllResponseHeaders())},b.async?4===g.readyState?a.setTimeout(c):g.onreadystatechange=fc[h]=c:c()},abort:function(){c&&c(void 0,!0)}}}});function hc(){try{return new a.XMLHttpRequest}catch(b){}}function ic(){try{return new a.ActiveXObject("Microsoft.XMLHTTP")}catch(b){}}n.ajaxSetup({accepts:{script:"text/javascript, application/javascript, application/ecmascript, application/x-ecmascript"},contents:{script:/\b(?:java|ecma)script\b/},converters:{"text script":function(a){return n.globalEval(a),a}}}),n.ajaxPrefilter("script",function(a){void 0===a.cache&&(a.cache=!1),a.crossDomain&&(a.type="GET",a.global=!1)}),n.ajaxTransport("script",function(a){if(a.crossDomain){var b,c=d.head||n("head")[0]||d.documentElement;return{send:function(e,f){b=d.createElement("script"),b.async=!0,a.scriptCharset&&(b.charset=a.scriptCharset),b.src=a.url,b.onload=b.onreadystatechange=function(a,c){(c||!b.readyState||/loaded|complete/.test(b.readyState))&&(b.onload=b.onreadystatechange=null,b.parentNode&&b.parentNode.removeChild(b),b=null,c||f(200,"success"))},c.insertBefore(b,c.firstChild)},abort:function(){b&&b.onload(void 0,!0)}}}});var jc=[],kc=/(=)\?(?=&|$)|\?\?/;n.ajaxSetup({jsonp:"callback",jsonpCallback:function(){var a=jc.pop()||n.expando+"_"+Eb++;return this[a]=!0,a}}),n.ajaxPrefilter("json jsonp",function(b,c,d){var e,f,g,h=b.jsonp!==!1&&(kc.test(b.url)?"url":"string"==typeof b.data&&0===(b.contentType||"").indexOf("application/x-www-form-urlencoded")&&kc.test(b.data)&&"data");return h||"jsonp"===b.dataTypes[0]?(e=b.jsonpCallback=n.isFunction(b.jsonpCallback)?b.jsonpCallback():b.jsonpCallback,h?b[h]=b[h].replace(kc,"$1"+e):b.jsonp!==!1&&(b.url+=(Fb.test(b.url)?"&":"?")+b.jsonp+"="+e),b.converters["script json"]=function(){return g||n.error(e+" was not called"),g[0]},b.dataTypes[0]="json",f=a[e],a[e]=function(){g=arguments},d.always(function(){void 0===f?n(a).removeProp(e):a[e]=f,b[e]&&(b.jsonpCallback=c.jsonpCallback,jc.push(e)),g&&n.isFunction(f)&&f(g[0]),g=f=void 0}),"script"):void 0}),n.parseHTML=function(a,b,c){if(!a||"string"!=typeof a)return null;"boolean"==typeof b&&(c=b,b=!1),b=b||d;var e=x.exec(a),f=!c&&[];return e?[b.createElement(e[1])]:(e=ja([a],b,f),f&&f.length&&n(f).remove(),n.merge([],e.childNodes))};var lc=n.fn.load;n.fn.load=function(a,b,c){if("string"!=typeof a&&lc)return lc.apply(this,arguments);var d,e,f,g=this,h=a.indexOf(" ");return h>-1&&(d=n.trim(a.slice(h,a.length)),a=a.slice(0,h)),n.isFunction(b)?(c=b,b=void 0):b&&"object"==typeof b&&(e="POST"),g.length>0&&n.ajax({url:a,type:e||"GET",dataType:"html",data:b}).done(function(a){f=arguments,g.html(d?n("<div>").append(n.parseHTML(a)).find(d):a)}).always(c&&function(a,b){g.each(function(){c.apply(this,f||[a.responseText,b,a])})}),this},n.each(["ajaxStart","ajaxStop","ajaxComplete","ajaxError","ajaxSuccess","ajaxSend"],function(a,b){n.fn[b]=function(a){return this.on(b,a)}}),n.expr.filters.animated=function(a){return n.grep(n.timers,function(b){return a===b.elem}).length};function mc(a){return n.isWindow(a)?a:9===a.nodeType?a.defaultView||a.parentWindow:!1}n.offset={setOffset:function(a,b,c){var d,e,f,g,h,i,j,k=n.css(a,"position"),l=n(a),m={};"static"===k&&(a.style.position="relative"),h=l.offset(),f=n.css(a,"top"),i=n.css(a,"left"),j=("absolute"===k||"fixed"===k)&&n.inArray("auto",[f,i])>-1,j?(d=l.position(),g=d.top,e=d.left):(g=parseFloat(f)||0,e=parseFloat(i)||0),n.isFunction(b)&&(b=b.call(a,c,n.extend({},h))),null!=b.top&&(m.top=b.top-h.top+g),null!=b.left&&(m.left=b.left-h.left+e),"using"in b?b.using.call(a,m):l.css(m)}},n.fn.extend({offset:function(a){if(arguments.length)return void 0===a?this:this.each(function(b){n.offset.setOffset(this,a,b)});var b,c,d={top:0,left:0},e=this[0],f=e&&e.ownerDocument;if(f)return b=f.documentElement,n.contains(b,e)?("undefined"!=typeof e.getBoundingClientRect&&(d=e.getBoundingClientRect()),c=mc(f),{top:d.top+(c.pageYOffset||b.scrollTop)-(b.clientTop||0),left:d.left+(c.pageXOffset||b.scrollLeft)-(b.clientLeft||0)}):d},position:function(){if(this[0]){var a,b,c={top:0,left:0},d=this[0];return"fixed"===n.css(d,"position")?b=d.getBoundingClientRect():(a=this.offsetParent(),b=this.offset(),n.nodeName(a[0],"html")||(c=a.offset()),c.top+=n.css(a[0],"borderTopWidth",!0),c.left+=n.css(a[0],"borderLeftWidth",!0)),{top:b.top-c.top-n.css(d,"marginTop",!0),left:b.left-c.left-n.css(d,"marginLeft",!0)}}},offsetParent:function(){return this.map(function(){var a=this.offsetParent;while(a&&!n.nodeName(a,"html")&&"static"===n.css(a,"position"))a=a.offsetParent;return a||Qa})}}),n.each({scrollLeft:"pageXOffset",scrollTop:"pageYOffset"},function(a,b){var c=/Y/.test(b);n.fn[a]=function(d){return Y(this,function(a,d,e){var f=mc(a);return void 0===e?f?b in f?f[b]:f.document.documentElement[d]:a[d]:void(f?f.scrollTo(c?n(f).scrollLeft():e,c?e:n(f).scrollTop()):a[d]=e)},a,d,arguments.length,null)}}),n.each(["top","left"],function(a,b){n.cssHooks[b]=Ua(l.pixelPosition,function(a,c){return c?(c=Sa(a,b),Oa.test(c)?n(a).position()[b]+"px":c):void 0})}),n.each({Height:"height",Width:"width"},function(a,b){n.each({
padding:"inner"+a,content:b,"":"outer"+a},function(c,d){n.fn[d]=function(d,e){var f=arguments.length&&(c||"boolean"!=typeof d),g=c||(d===!0||e===!0?"margin":"border");return Y(this,function(b,c,d){var e;return n.isWindow(b)?b.document.documentElement["client"+a]:9===b.nodeType?(e=b.documentElement,Math.max(b.body["scroll"+a],e["scroll"+a],b.body["offset"+a],e["offset"+a],e["client"+a])):void 0===d?n.css(b,c,g):n.style(b,c,d,g)},b,f?d:void 0,f,null)}})}),n.fn.extend({bind:function(a,b,c){return this.on(a,null,b,c)},unbind:function(a,b){return this.off(a,null,b)},delegate:function(a,b,c,d){return this.on(b,a,c,d)},undelegate:function(a,b,c){return 1===arguments.length?this.off(a,"**"):this.off(b,a||"**",c)}}),n.fn.size=function(){return this.length},n.fn.andSelf=n.fn.addBack,"function"==typeof define&&define.amd&&define("jquery",[],function(){return n});var nc=a.jQuery,oc=a.$;return n.noConflict=function(b){return a.$===n&&(a.$=oc),b&&a.jQuery===n&&(a.jQuery=nc),n},b||(a.jQuery=a.$=n),n});
</script>
<style type="text/css">.dt-crosstalk-fade {
opacity: 0.2;
}
html body div.DTS div.dataTables_scrollBody {
background: none;
}
</style>
<script>(function() {

// some helper functions: using a global object DTWidget so that it can be used
// in JS() code, e.g. datatable(options = list(foo = JS('code'))); unlike R's
// dynamic scoping, when 'code' is eval()'ed, JavaScript does not know objects
// from the "parent frame", e.g. JS('DTWidget') will not work unless it was made
// a global object
var DTWidget = {};

// 123456666.7890 -> 123,456,666.7890
var markInterval = function(d, digits, interval, mark, decMark, precision) {
  x = precision ? d.toPrecision(digits) : d.toFixed(digits);
  if (!/^-?[\d.]+$/.test(x)) return x;
  var xv = x.split('.');
  if (xv.length > 2) return x;  // should have at most one decimal point
  xv[0] = xv[0].replace(new RegExp('\\B(?=(\\d{' + interval + '})+(?!\\d))', 'g'), mark);
  return xv.join(decMark);
};

DTWidget.formatCurrency = function(thiz, row, data, col, currency, digits, interval, mark, decMark, before) {
  var d = parseFloat(data[col]);
  if (isNaN(d)) return;
  var res = markInterval(d, digits, interval, mark, decMark);
  res = before ? (/^-/.test(res) ? '-' + currency + res.replace(/^-/, '') : currency + res) :
    res + currency;
  $(thiz.api().cell(row, col).node()).html(res);
};

DTWidget.formatString = function(thiz, row, data, col, prefix, suffix) {
  var d = data[col];
  if (d === null) return;
  var cell = $(thiz.api().cell(row, col).node());
  cell.html(prefix + cell.html() + suffix);
};

DTWidget.formatPercentage = function(thiz, row, data, col, digits, interval, mark, decMark) {
  var d = parseFloat(data[col]);
  if (isNaN(d)) return;
  $(thiz.api().cell(row, col).node())
  .html(markInterval(d * 100, digits, interval, mark, decMark) + '%');
};

DTWidget.formatRound = function(thiz, row, data, col, digits, interval, mark, decMark) {
  var d = parseFloat(data[col]);
  if (isNaN(d)) return;
  $(thiz.api().cell(row, col).node()).html(markInterval(d, digits, interval, mark, decMark));
};

DTWidget.formatSignif = function(thiz, row, data, col, digits, interval, mark, decMark) {
  var d = parseFloat(data[col]);
  if (isNaN(d)) return;
  $(thiz.api().cell(row, col).node())
    .html(markInterval(d, digits, interval, mark, decMark, true));
};

DTWidget.formatDate = function(thiz, row, data, col, method, params) {
  var d = data[col];
  if (d === null) return;
  // (new Date('2015-10-28')).toDateString() may return 2015-10-27 because the
  // actual time created could be like 'Tue Oct 27 2015 19:00:00 GMT-0500 (CDT)',
  // i.e. the date-only string is treated as UTC time instead of local time
  if ((method === 'toDateString' || method === 'toLocaleDateString') && /^\d{4,}\D\d{2}\D\d{2}$/.test(d)) {
    d = d.split(/\D/);
    d = new Date(d[0], d[1] - 1, d[2]);
  } else {
    d = new Date(d);
  }
  $(thiz.api().cell(row, col).node()).html(d[method].apply(d, params));
};

window.DTWidget = DTWidget;

var transposeArray2D = function(a) {
  return a.length === 0 ? a : HTMLWidgets.transposeArray2D(a);
};

var crosstalkPluginsInstalled = false;

function maybeInstallCrosstalkPlugins() {
  if (crosstalkPluginsInstalled)
    return;
  crosstalkPluginsInstalled = true;

  $.fn.dataTable.ext.afnFiltering.push(
    function(oSettings, aData, iDataIndex) {
      var ctfilter = oSettings.nTable.ctfilter;
      if (ctfilter && !ctfilter[iDataIndex])
        return false;

      var ctselect = oSettings.nTable.ctselect;
      if (ctselect && !ctselect[iDataIndex])
        return false;

      return true;
    }
  );
}

HTMLWidgets.widget({
  name: "datatables",
  type: "output",
  renderOnNullValue: true,
  initialize: function(el, width, height) {
    $(el).html('&nbsp;');
    return {
      data: null,
      ctfilterHandle: new crosstalk.FilterHandle(),
      ctfilterSubscription: null,
      ctselectHandle: new crosstalk.SelectionHandle(),
      ctselectSubscription: null
    };
  },
  renderValue: function(el, data, instance) {
    if (el.offsetWidth === 0 || el.offsetHeight === 0) {
      instance.data = data;
      return;
    }
    instance.data = null;
    var $el = $(el);
    $el.empty();

    if (data === null) {
      $el.append('&nbsp;');
      // clear previous Shiny inputs (if any)
      for (var i in instance.clearInputs) instance.clearInputs[i]();
      instance.clearInputs = {};
      return;
    }

    var crosstalkOptions = data.crosstalkOptions;
    if (!crosstalkOptions) crosstalkOptions = {
      'key': null, 'group': null
    };
    if (crosstalkOptions.group) {
      maybeInstallCrosstalkPlugins();
      instance.ctfilterHandle.setGroup(crosstalkOptions.group);
      instance.ctselectHandle.setGroup(crosstalkOptions.group);
    }

    // If we are in a flexdashboard scroll layout then we:
    //  (a) Always want to use pagination (otherwise we'll have
    //      a "double scroll bar" effect on the phone); and
    //  (b) Never want to fill the container (we want the pagination
    //      level to determine the size of the container)
    if (window.FlexDashboard && !window.FlexDashboard.isFillPage()) {
      data.options.bPaginate = true;
      data.fillContainer = false;
    }

    // if we are in the viewer then we always want to fillContainer and
    // and autoHideNavigation (unless the user has explicitly set these)
    if (window.HTMLWidgets.viewerMode) {
      if (!data.hasOwnProperty("fillContainer"))
        data.fillContainer = true;
      if (!data.hasOwnProperty("autoHideNavigation"))
        data.autoHideNavigation = true;
    }

    // propagate fillContainer to instance (so we have it in resize)
    instance.fillContainer = data.fillContainer;

    var cells = data.data;

    if (cells instanceof Array) cells = transposeArray2D(cells);

    $el.append(data.container);
    var $table = $el.find('table');
    if (data.class) $table.addClass(data.class);
    if (data.caption) $table.prepend(data.caption);

    if (!data.selection) data.selection = {
      mode: 'none', selected: null, target: 'row'
    };
    if (HTMLWidgets.shinyMode && data.selection.mode !== 'none' &&
        data.selection.target === 'row+column') {
      if ($table.children('tfoot').length === 0) {
        $table.append($('<tfoot>'));
        $table.find('thead tr').clone().appendTo($table.find('tfoot'));
      }
    }

    // column filters
    var filterRow;
    switch (data.filter) {
      case 'top':
        $table.children('thead').append(data.filterHTML);
        filterRow = $table.find('thead tr:last td');
        break;
      case 'bottom':
        if ($table.children('tfoot').length === 0) {
          $table.append($('<tfoot>'));
        }
        $table.children('tfoot').prepend(data.filterHTML);
        filterRow = $table.find('tfoot tr:first td');
        break;
    }

    var options = { searchDelay: 1000 };
    if (cells !== null) $.extend(options, {
      data: cells
    });

    // options for fillContainer
    var bootstrapActive = typeof($.fn.popover) != 'undefined';
    if (instance.fillContainer) {

      // force scrollX/scrollY and turn off autoWidth
      options.scrollX = true;
      options.scrollY = "100px"; // can be any value, we'll adjust below

      // if we aren't paginating then move around the info/filter controls
      // to save space at the bottom and rephrase the info callback
      if (data.options.bPaginate === false) {

        // we know how to do this cleanly for bootstrap, not so much
        // for other themes/layouts
        if (bootstrapActive) {
          options.dom = "<'row'<'col-sm-4'i><'col-sm-8'f>>" +
                        "<'row'<'col-sm-12'tr>>";
        }

        options.fnInfoCallback = function(oSettings, iStart, iEnd,
                                           iMax, iTotal, sPre) {
          return Number(iTotal).toLocaleString() + " records";
        };
      }
    }

    // auto hide navigation if requested
    if (data.autoHideNavigation === true) {
      if (bootstrapActive && data.options.bPaginate !== false) {
        // strip all nav if length >= cells
        if ((cells instanceof Array) && data.options.iDisplayLength >= cells.length)
          options.dom = "<'row'<'col-sm-12'tr>>";
        // alternatively lean things out for flexdashboard mobile portrait
        else if (window.FlexDashboard && window.FlexDashboard.isMobilePhone())
          options.dom = "<'row'<'col-sm-12'f>>" +
                        "<'row'<'col-sm-12'tr>>"  +
                        "<'row'<'col-sm-12'p>>";
      }
    }

    $.extend(true, options, data.options || {});

    var searchCols = options.searchCols;
    if (searchCols) {
      searchCols = searchCols.map(function(x) {
        return x === null ? '' : x.search;
      });
      // FIXME: this means I don't respect the escapeRegex setting
      delete options.searchCols;
    }

    // server-side processing?
    var server = options.serverSide === true;

    // use the dataSrc function to pre-process JSON data returned from R
    var DT_rows_all = [], DT_rows_current = [];
    if (server && HTMLWidgets.shinyMode && typeof options.ajax === 'object' &&
        /^session\/[\da-z]+\/dataobj/.test(options.ajax.url) && !options.ajax.dataSrc) {
      options.ajax.dataSrc = function(json) {
        DT_rows_all = $.makeArray(json.DT_rows_all);
        DT_rows_current = $.makeArray(json.DT_rows_current);
        var data = json.data;
        if (!colReorderEnabled()) return data;
        var table = $table.DataTable(), order = table.colReorder.order(), flag = true, i, j, row;
        for (i = 0; i < order.length; ++i) if (order[i] !== i) flag = false;
        if (flag) return data;
        for (i = 0; i < data.length; ++i) {
          row = data[i].slice();
          for (j = 0; j < order.length; ++j) data[i][j] = row[order[j]];
        }
        return data;
      };
    }

    var thiz = this;
    if (instance.fillContainer) $table.on('init.dt', function(e) {
      thiz.fillAvailableHeight(el, $(el).innerHeight());
    });
    // If the page contains serveral datatables and one of which enables colReorder,
    // the table.colReorder.order() function will exist but throws error when called.
    // So it seems like the only way to know if colReorder is enabled or not is to
    // check the options.
    var colReorderEnabled = function() { return "colReorder" in options; };
    var table = $table.DataTable(options);
    $el.data('datatable', table);

    // Unregister previous Crosstalk event subscriptions, if they exist
    if (instance.ctfilterSubscription) {
      instance.ctfilterHandle.off("change", instance.ctfilterSubscription);
      instance.ctfilterSubscription = null;
    }
    if (instance.ctselectSubscription) {
      instance.ctselectHandle.off("change", instance.ctselectSubscription);
      instance.ctselectSubscription = null;
    }

    if (!crosstalkOptions.group) {
      $table[0].ctfilter = null;
      $table[0].ctselect = null;
    } else {
      var key = crosstalkOptions.key;
      function keysToMatches(keys) {
        if (!keys) {
          return null;
        } else {
          var selectedKeys = {};
          for (var i = 0; i < keys.length; i++) {
            selectedKeys[keys[i]] = true;
          }
          var matches = {};
          for (var j = 0; j < key.length; j++) {
            if (selectedKeys[key[j]])
              matches[j] = true;
          }
          return matches;
        }
      }

      function applyCrosstalkFilter(e) {
        $table[0].ctfilter = keysToMatches(e.value);
        table.draw();
      }
      instance.ctfilterSubscription = instance.ctfilterHandle.on("change", applyCrosstalkFilter);
      applyCrosstalkFilter({value: instance.ctfilterHandle.filteredKeys});

      function applyCrosstalkSelection(e) {
        if (e.sender !== instance.ctselectHandle) {
          table
            .rows('.' + selClass, {search: 'applied'})
            .nodes()
            .to$()
            .removeClass(selClass);
          if (selectedRows)
            changeInput('rows_selected', selectedRows(), void 0, true);
        }

        if (e.sender !== instance.ctselectHandle && e.value && e.value.length) {
          var matches = keysToMatches(e.value);

          // persistent selection with plotly (& leaflet)
          var ctOpts = crosstalk.var("plotlyCrosstalkOpts").get() || {};
          if (ctOpts.persistent === true) {
            var matches = $.extend(matches, $table[0].ctselect);
          }

          $table[0].ctselect = matches;
          table.draw();
        } else {
          if ($table[0].ctselect) {
            $table[0].ctselect = null;
            table.draw();
          }
        }
      }
      instance.ctselectSubscription = instance.ctselectHandle.on("change", applyCrosstalkSelection);
      // TODO: This next line doesn't seem to work when renderDataTable is used
      applyCrosstalkSelection({value: instance.ctselectHandle.value});
    }

    var inArray = function(val, array) {
      return $.inArray(val, $.makeArray(array)) > -1;
    };

    // encode + to %2B when searching in the table on server side, because
    // shiny::parseQueryString() treats + as spaces, and DataTables does not
    // encode + to %2B (or % to %25) when sending the request
    var encode_plus = function(x) {
      return server ? x.replace(/%/g, '%25').replace(/\+/g, '%2B') : x;
    };

    // search the i-th column
    var searchColumn = function(i, value) {
      var regex = false, ci = true;
      if (options.search) {
        regex = options.search.regex,
        ci = options.search.caseInsensitive !== false;
      }
      return table.column(i).search(encode_plus(value), regex, !regex, ci);
    };

    if (data.filter !== 'none') {

      filterRow.each(function(i, td) {

        var $td = $(td), type = $td.data('type'), filter;
        var $input = $td.children('div').first().children('input');
        $input.prop('disabled', !table.settings()[0].aoColumns[i].bSearchable || type === 'disabled');
        $input.on('input blur', function() {
          $input.next('span').toggle(Boolean($input.val()));
        });
        // Bootstrap sets pointer-events to none and we won't be able to click
        // the clear button
        $input.next('span').css('pointer-events', 'auto').hide().click(function() {
          $(this).hide().prev('input').val('').trigger('input').focus();
        });
        var searchCol;  // search string for this column
        if (searchCols && searchCols[i]) {
          searchCol = searchCols[i];
          $input.val(searchCol).trigger('input');
        }
        var $x = $td.children('div').last();

        // remove the overflow: hidden attribute of the scrollHead
        // (otherwise the scrolling table body obscures the filters)
        var scrollHead = $(el).find('.dataTables_scrollHead,.dataTables_scrollFoot');
        var cssOverflow = scrollHead.css('overflow');
        if (cssOverflow === 'hidden') {
          $x.on('show hide', function(e) {
            scrollHead.css('overflow', e.type === 'show' ? '' : cssOverflow);
          });
          $x.css('z-index', 25);
        }

        if (inArray(type, ['factor', 'logical'])) {
          $input.on({
            click: function() {
              $input.parent().hide(); $x.show().trigger('show'); filter[0].selectize.focus();
            },
            input: function() {
              if ($input.val() === '') filter[0].selectize.setValue([]);
            }
          });
          var $input2 = $x.children('select');
          filter = $input2.selectize({
            options: $input2.data('options').map(function(v, i) {
              return ({text: v, value: v});
            }),
            plugins: ['remove_button'],
            hideSelected: true,
            onChange: function(value) {
              if (value === null) value = []; // compatibility with jQuery 3.0
              $input.val(value.length ? JSON.stringify(value) : '');
              if (value.length) $input.trigger('input');
              $input.attr('title', $input.val());
              if (server) {
                table.column(i).search(value.length ? encode_plus(JSON.stringify(value)) : '').draw();
                return;
              }
              // turn off filter if nothing selected
              $td.data('filter', value.length > 0);
              table.draw();  // redraw table, and filters will be applied
            }
          });
          if (searchCol) filter[0].selectize.setValue(JSON.parse(searchCol));
          filter[0].selectize.on('blur', function() {
            $x.hide().trigger('hide'); $input.parent().show(); $input.trigger('blur');
          });
          filter.next('div').css('margin-bottom', 'auto');
        } else if (type === 'character') {
          var fun = function() {
            searchColumn(i, $input.val()).draw();
          };
          if (server) {
            fun = $.fn.dataTable.util.throttle(fun, options.searchDelay);
          }
          $input.on('input', fun);
        } else if (inArray(type, ['number', 'integer', 'date', 'time'])) {
          var $x0 = $x;
          $x = $x0.children('div').first();
          $x0.css({
            'background-color': '#fff',
            'border': '1px #ddd solid',
            'border-radius': '4px',
            'padding': '20px 20px 10px 20px'
          });
          var $spans = $x0.children('span').css({
            'margin-top': '10px',
            'white-space': 'nowrap'
          });
          var $span1 = $spans.first(), $span2 = $spans.last();
          var r1 = +$x.data('min'), r2 = +$x.data('max');
          // when the numbers are too small or have many decimal places, the
          // slider may have numeric precision problems (#150)
          var scale = Math.pow(10, Math.max(0, +$x.data('scale') || 0));
          r1 = Math.round(r1 * scale); r2 = Math.round(r2 * scale);
          var scaleBack = function(x, scale) {
            if (scale === 1) return x;
            var d = Math.round(Math.log(scale) / Math.log(10));
            // to avoid problems like 3.423/100 -> 0.034230000000000003
            return (x / scale).toFixed(d);
          };
          $input.on({
            focus: function() {
              $x0.show().trigger('show');
              // first, make sure the slider div leaves at least 20px between
              // the two (slider value) span's
              $x0.width(Math.max(160, $span1.outerWidth() + $span2.outerWidth() + 20));
              // then, if the input is really wide, make the slider the same
              // width as the input
              if ($x0.outerWidth() < $input.outerWidth()) {
                $x0.outerWidth($input.outerWidth());
              }
              // make sure the slider div does not reach beyond the right margin
              if ($(window).width() < $x0.offset().left + $x0.width()) {
                $x0.offset({
                  'left': $input.offset().left + $input.outerWidth() - $x0.outerWidth()
                });
              }
            },
            blur: function() {
              $x0.hide().trigger('hide');
            },
            input: function() {
              if ($input.val() === '') filter.val([r1, r2]);
            },
            change: function() {
              var v = $input.val().replace(/\s/g, '');
              if (v === '') return;
              v = v.split('...');
              if (v.length !== 2) {
                $input.parent().addClass('has-error');
                return;
              }
              if (v[0] === '') v[0] = r1;
              if (v[1] === '') v[1] = r2;
              $input.parent().removeClass('has-error');
              // treat date as UTC time at midnight
              var strTime = function(x) {
                var s = type === 'date' ? 'T00:00:00Z' : '';
                var t = new Date(x + s).getTime();
                // add 10 minutes to date since it does not hurt the date, and
                // it helps avoid the tricky floating point arithmetic problems,
                // e.g. sometimes the date may be a few milliseconds earlier
                // than the midnight due to precision problems in noUiSlider
                return type === 'date' ? t + 3600000 : t;
              };
              if (inArray(type, ['date', 'time'])) {
                v[0] = strTime(v[0]);
                v[1] = strTime(v[1]);
              }
              if (v[0] != r1) v[0] *= scale;
              if (v[1] != r2) v[1] *= scale;
              filter.val(v);
            }
          });
          var formatDate = function(d, isoFmt) {
            d = scaleBack(d, scale);
            if (type === 'number') return d;
            if (type === 'integer') return parseInt(d);
            var x = new Date(+d);
            var fmt = ('filterDateFmt' in data) ? data.filterDateFmt[i] : undefined;
            if (fmt !== undefined && isoFmt === false) return x[fmt.method].apply(x, fmt.params);
            if (type === 'date') {
              var pad0 = function(x) {
                return ('0' + x).substr(-2, 2);
              };
              return x.getUTCFullYear() + '-' + pad0(1 + x.getUTCMonth())
                      + '-' + pad0(x.getUTCDate());
            } else {
              return x.toISOString();
            }
          };
          var opts = type === 'date' ? { step: 60 * 60 * 1000 } :
                     type === 'integer' ? { step: 1 } : {};
          filter = $x.noUiSlider($.extend({
            start: [r1, r2],
            range: {min: r1, max: r2},
            connect: true
          }, opts));
          if (scale > 1) (function() {
            var t1 = r1, t2 = r2;
            var val = filter.val();
            while (val[0] > r1 || val[1] < r2) {
              if (val[0] > r1) {
                t1 -= val[0] - r1;
              }
              if (val[1] < r2) {
                t2 += r2 - val[1];
              }
              filter = $x.noUiSlider($.extend({
                start: [t1, t2],
                range: {min: t1, max: t2},
                connect: true
              }, opts), true);
              val = filter.val();
            }
            r1  = t1; r2 = t2;
          })();
          var updateSliderText = function(v1, v2) {
            $span1.text(formatDate(v1, false)); $span2.text(formatDate(v2, false));
          };
          updateSliderText(r1, r2);
          var updateSlider = function(e) {
            var val = filter.val();
            // turn off filter if in full range
            $td.data('filter', val[0] > r1 || val[1] < r2);
            var v1 = formatDate(val[0]), v2 = formatDate(val[1]), ival;
            if ($td.data('filter')) {
              ival = v1 + ' ... ' + v2;
              $input.attr('title', ival).val(ival).trigger('input');
            } else {
              $input.attr('title', '').val('');
            }
            updateSliderText(val[0], val[1]);
            if (e.type === 'slide') return;  // no searching when sliding only
            if (server) {
              table.column(i).search($td.data('filter') ? ival : '').draw();
              return;
            }
            table.draw();
          };
          filter.on({
            set: updateSlider,
            slide: updateSlider
          });
        }

        // server-side processing will be handled by R (or whatever server
        // language you use); the following code is only needed for client-side
        // processing
        if (server) {
          // if a search string has been pre-set, search now
          if (searchCol) searchColumn(i, searchCol).draw();
          return;
        }

        var customFilter = function(settings, data, dataIndex) {
          // there is no way to attach a search function to a specific table,
          // and we need to make sure a global search function is not applied to
          // all tables (i.e. a range filter in a previous table should not be
          // applied to the current table); we use the settings object to
          // determine if we want to perform searching on the current table,
          // since settings.sTableId will be different to different tables
          if (table.settings()[0] !== settings) return true;
          // no filter on this column or no need to filter this column
          if (typeof filter === 'undefined' || !$td.data('filter')) return true;

          var r = filter.val(), v, r0, r1;
          var i_data = function(i) {
            if (!colReorderEnabled()) return i;
            var order = table.colReorder.order(), k;
            for (k = 0; k < order.length; ++k) if (order[k] === i) return k;
            return i; // in theory it will never be here...
          }
          v = data[i_data(i)];
          if (type === 'number' || type === 'integer') {
            v = parseFloat(v);
            // how to handle NaN? currently exclude these rows
            if (isNaN(v)) return(false);
            r0 = parseFloat(scaleBack(r[0], scale))
            r1 = parseFloat(scaleBack(r[1], scale));
            if (v >= r0 && v <= r1) return true;
          } else if (type === 'date' || type === 'time') {
            v = new Date(v);
            r0 = new Date(r[0] / scale); r1 = new Date(r[1] / scale);
            if (v >= r0 && v <= r1) return true;
          } else if (type === 'factor') {
            if (r.length === 0 || inArray(v, r)) return true;
          } else if (type === 'logical') {
            if (r.length === 0) return true;
            if (inArray(v === '' ? 'na' : v, r)) return true;
          }
          return false;
        };

        $.fn.dataTable.ext.search.push(customFilter);

        // search for the preset search strings if it is non-empty
        if (searchCol) {
          if (inArray(type, ['factor', 'logical'])) {
            filter[0].selectize.setValue(JSON.parse(searchCol));
          } else if (type === 'character') {
            $input.trigger('input');
          } else if (inArray(type, ['number', 'integer', 'date', 'time'])) {
            $input.trigger('change');
          }
        }

      });

    }

    // highlight search keywords
    var highlight = function() {
      var body = $(table.table().body());
      // removing the old highlighting first
      body.unhighlight();

      // don't highlight the "not found" row, so we get the rows using the api
      if (table.rows({ filter: 'applied' }).data().length === 0) return;
      // highlight gloal search keywords
      body.highlight($.trim(table.search()).split(/\s+/));
      // then highlight keywords from individual column filters
      if (filterRow) filterRow.each(function(i, td) {
        var $td = $(td), type = $td.data('type');
        if (type !== 'character') return;
        var $input = $td.children('div').first().children('input');
        var column = table.column(i).nodes().to$(),
            val = $.trim($input.val());
        if (type !== 'character' || val === '') return;
        column.highlight(val.split(/\s+/));
      });
    };

    if (options.searchHighlight) {
      table
      .on('draw.dt.dth column-visibility.dt.dth column-reorder.dt.dth', highlight)
      .on('destroy', function() {
        // remove event handler
        table.off('draw.dt.dth column-visibility.dt.dth column-reorder.dt.dth');
      });

      // initial highlight for state saved conditions and initial states
      highlight();
    }

    // run the callback function on the table instance
    if (typeof data.callback === 'function') data.callback(table);

    // double click to edit the cell
    if (data.editable) table.on('dblclick.dt', 'tbody td', function() {
      var $input = $('<input type="text">');
      var $this = $(this), value = table.cell(this).data(), html = $this.html();
      var changed = false;
      $input.val(value);
      $this.empty().append($input);
      $input.css('width', '100%').focus().on('change', function() {
        changed = true;
        var valueNew = $input.val();
        if (valueNew != value) {
          table.cell($this).data(valueNew);
          if (HTMLWidgets.shinyMode) changeInput('cell_edit', cellInfo($this));
          // for server-side processing, users have to call replaceData() to update the table
          if (!server) table.draw(false);
        } else {
          $this.html(html);
        }
        $input.remove();
      }).on('blur', function() {
        if (!changed) $input.trigger('change');
      });
    });

    // interaction with shiny
    if (!HTMLWidgets.shinyMode && !crosstalkOptions.group) return;

    var methods = {};
    var shinyData = {};

    methods.updateCaption = function(caption) {
      if (!caption) return;
      $table.children('caption').replaceWith(caption);
    }

    // register clear functions to remove input values when the table is removed
    instance.clearInputs = {};

    var changeInput = function(id, value, type, noCrosstalk) {
      var event = id;
      id = el.id + '_' + id;
      if (type) id = id + ':' + type;
      // do not update if the new value is the same as old value
      if (shinyData.hasOwnProperty(id) && shinyData[id] === JSON.stringify(value))
        return;
      shinyData[id] = JSON.stringify(value);
      if (HTMLWidgets.shinyMode) {
        Shiny.onInputChange(id, value);
        if (!instance.clearInputs[id]) instance.clearInputs[id] = function() {
          Shiny.onInputChange(id, null);
        }
      }

      // HACK
      if (event === "rows_selected" && !noCrosstalk) {
        if (crosstalkOptions.group) {
          var keys = crosstalkOptions.key;
          var selectedKeys = null;
          if (value) {
            selectedKeys = [];
            for (var i = 0; i < value.length; i++) {
              // The value array's contents use 1-based row numbers, so we must
              // convert to 0-based before indexing into the keys array.
              selectedKeys.push(keys[value[i] - 1]);
            }
          }
          instance.ctselectHandle.set(selectedKeys);
        }
      }
    };

    var addOne = function(x) {
      return x.map(function(i) { return 1 + i; });
    };

    var unique = function(x) {
      var ux = [];
      $.each(x, function(i, el){
        if ($.inArray(el, ux) === -1) ux.push(el);
      });
      return ux;
    }

    // change the row index of a cell
    var tweakCellIndex = function(cell) {
      var info = cell.index();
      if (server) {
        info.row = DT_rows_current[info.row];
      } else {
        info.row += 1;
      }
      return {row: info.row, col: info.column};
    }

    var selMode = data.selection.mode, selTarget = data.selection.target;
    if (inArray(selMode, ['single', 'multiple'])) {
      var selClass = data.style === 'bootstrap' ? 'active' : 'selected';
      var selected = data.selection.selected, selected1, selected2;
      // selected1: row indices; selected2: column indices
      if (selected === null) {
        selected1 = selected2 = [];
      } else if (selTarget === 'row') {
        selected1 = $.makeArray(selected);
      } else if (selTarget === 'column') {
        selected2 = $.makeArray(selected);
      } else if (selTarget === 'row+column') {
        selected1 = $.makeArray(selected.rows);
        selected2 = $.makeArray(selected.cols);
      }

      // After users reorder the rows or filter the table, we cannot use the table index
      // directly. Instead, we need this function to find out the rows between the two clicks.
      // If user filter the table again between the start click and the end click, the behavior
      // would be undefined, but it should not be a problem.
      var shiftSelRowsIndex = function(start, end) {
        var indexes = server ? DT_rows_all : table.rows({ search: 'applied' }).indexes().toArray();
        start = indexes.indexOf(start); end = indexes.indexOf(end);
        // if start is larger than end, we need to swap
        if (start > end) {
          var tmp = end; end = start; start = tmp;
        }
        return indexes.slice(start, end + 1);
      }

      var serverRowIndex = function(clientRowIndex) {
        return server ? DT_rows_current[clientRowIndex] : clientRowIndex + 1;
      }

      // row, column, or cell selection
      var lastClickedRow;
      if (inArray(selTarget, ['row', 'row+column'])) {
        var selectedRows = function() {
          var rows = table.rows('.' + selClass);
          var idx = rows.indexes().toArray();
          if (!server) return addOne(idx);
          idx = idx.map(function(i) {
            return DT_rows_current[i];
          });
          selected1 = selMode === 'multiple' ? unique(selected1.concat(idx)) : idx;
          return selected1;
        }
        table.on('mousedown.dt', 'tbody tr', function(e) {
          var $this = $(this), thisRow = table.row(this);
          if (selMode === 'multiple') {
            if (e.shiftKey && lastClickedRow !== undefined) {
              // select or de-select depends on the last clicked row's status
              var flagSel = !$this.hasClass(selClass);
              var crtClickedRow = serverRowIndex(thisRow.index());
              if (server) {
                var rowsIndex = shiftSelRowsIndex(lastClickedRow, crtClickedRow);
                // update current page's selClass
                rowsIndex.map(function(i) {
                  var rowIndex = DT_rows_current.indexOf(i);
                  if (rowIndex >= 0) {
                    var row = table.row(rowIndex).nodes().to$();
                    var flagRowSel = !row.hasClass(selClass);
                    if (flagSel === flagRowSel) row.toggleClass(selClass);
                  }
                });
                // update selected1
                if (flagSel) {
                  selected1 = unique(selected1.concat(rowsIndex));
                } else {
                  selected1 = selected1.filter(function(index) {
                    return !inArray(index, rowsIndex);
                  });
                }
              } else {
                // js starts from 0
                shiftSelRowsIndex(lastClickedRow - 1, crtClickedRow - 1).map(function(value) {
                  var row = table.row(value).nodes().to$();
                  var flagRowSel = !row.hasClass(selClass);
                  if (flagSel === flagRowSel) row.toggleClass(selClass);
                });
              }
              e.preventDefault();
            } else {
              $this.toggleClass(selClass);
            }
          } else {
            if ($this.hasClass(selClass)) {
              $this.removeClass(selClass);
            } else {
              table.$('tr.' + selClass).removeClass(selClass);
              $this.addClass(selClass);
            }
          }
          if (server && !$this.hasClass(selClass)) {
            var id = DT_rows_current[thisRow.index()];
            // remove id from selected1 since its class .selected has been removed
            if (inArray(id, selected1)) selected1.splice($.inArray(id, selected1), 1);
          }
          changeInput('rows_selected', selectedRows());
          changeInput('row_last_clicked', serverRowIndex(thisRow.index()));
          lastClickedRow = serverRowIndex(thisRow.index());
        });
        changeInput('rows_selected', selected1);
        var selectRows = function() {
          table.$('tr.' + selClass).removeClass(selClass);
          if (selected1.length === 0) return;
          if (server) {
            table.rows({page: 'current'}).every(function() {
              if (inArray(DT_rows_current[this.index()], selected1)) {
                $(this.node()).addClass(selClass);
              }
            });
          } else {
            var selected0 = selected1.map(function(i) { return i - 1; });
            $(table.rows(selected0).nodes()).addClass(selClass);
          }
        }
        selectRows();  // in case users have specified pre-selected rows
        // restore selected rows after the table is redrawn (e.g. sort/search/page);
        // client-side tables will preserve the selections automatically; for
        // server-side tables, we have to *real* row indices are in `selected1`
        if (server) table.on('draw.dt', selectRows);
        methods.selectRows = function(selected) {
          selected1 = $.makeArray(selected);
          selectRows();
          changeInput('rows_selected', selected1);
        }
      }

      if (inArray(selTarget, ['column', 'row+column'])) {
        if (selTarget === 'row+column') {
          $(table.columns().footer()).css('cursor', 'pointer');
        }
        table.on('click.dt', selTarget === 'column' ? 'tbody td' : 'tfoot tr th', function() {
          var colIdx = selTarget === 'column' ? table.cell(this).index().column :
              $.inArray(this, table.columns().footer()),
              thisCol = $(table.column(colIdx).nodes());
          if (colIdx === -1) return;
          if (thisCol.hasClass(selClass)) {
            thisCol.removeClass(selClass);
            selected2.splice($.inArray(colIdx, selected2), 1);
          } else {
            if (selMode === 'single') $(table.cells().nodes()).removeClass(selClass);
            thisCol.addClass(selClass);
            selected2 = selMode === 'single' ? [colIdx] : unique(selected2.concat([colIdx]));
          }
          changeInput('columns_selected', selected2);
        });
        changeInput('columns_selected', selected2);
        var selectCols = function() {
          table.columns().nodes().flatten().to$().removeClass(selClass);
          if (selected2.length > 0)
            table.columns(selected2).nodes().flatten().to$().addClass(selClass);
        }
        selectCols();  // in case users have specified pre-selected columns
        if (server) table.on('draw.dt', selectCols);
        methods.selectColumns = function(selected) {
          selected2 = $.makeArray(selected);
          selectCols();
          changeInput('columns_selected', selected2);
        }
      }

      if (selTarget === 'cell') {
        var selected3;
        if (selected === null) {
          selected3 = [];
        } else {
          selected3 = selected;
        }
        var findIndex = function(ij) {
          for (var i = 0; i < selected3.length; i++) {
            if (ij[0] === selected3[i][0] && ij[1] === selected3[i][1]) return i;
          }
          return -1;
        }
        table.on('click.dt', 'tbody td', function() {
          var $this = $(this), info = tweakCellIndex(table.cell(this));
          if ($this.hasClass(selClass)) {
            $this.removeClass(selClass);
            selected3.splice(findIndex([info.row, info.col]), 1);
          } else {
            if (selMode === 'single') $(table.cells().nodes()).removeClass(selClass);
            $this.addClass(selClass);
            selected3 = selMode === 'single' ? [[info.row, info.col]] :
              unique(selected3.concat([[info.row, info.col]]));
          }
          changeInput('cells_selected', transposeArray2D(selected3), 'shiny.matrix');
        });
        changeInput('cells_selected', transposeArray2D(selected3), 'shiny.matrix');
        var selectCells = function() {
          table.$('td.' + selClass).removeClass(selClass);
          if (selected3.length === 0) return;
          if (server) {
            table.cells({page: 'current'}).every(function() {
              var info = tweakCellIndex(this);
              if (findIndex([info.row, info.col], selected3) > -1)
                $(this.node()).addClass(selClass);
            });
          } else {
            selected3.map(function(ij) {
              $(table.cell(ij[0] - 1, ij[1]).node()).addClass(selClass);
            });
          }
        };
        selectCells();  // in case users have specified pre-selected columns
        if (server) table.on('draw.dt', selectCells);
        methods.selectCells = function(selected) {
          selected3 = selected ? selected : [];
          selectCells();
          changeInput('cells_selected', transposeArray2D(selected3), 'shiny.matrix');
        }
      }
    }

    // expose some table info to Shiny
    var updateTableInfo = function(e, settings) {
      // TODO: is anyone interested in the page info?
      // changeInput('page_info', table.page.info());
      var updateRowInfo = function(id, modifier) {
        var idx;
        if (server) {
          idx = modifier.page === 'current' ? DT_rows_current : DT_rows_all;
        } else {
          var rows = table.rows($.extend({
            search: 'applied',
            page: 'all'
          }, modifier));
          idx = addOne(rows.indexes().toArray());
        }
        changeInput('rows' + '_' + id, idx);
      };
      updateRowInfo('current', {page: 'current'});
      updateRowInfo('all', {});
    }
    table.on('draw.dt', updateTableInfo);
    updateTableInfo();

    // state info
    table.on('draw.dt column-visibility.dt', function() {
      changeInput('state', table.state());
    });
    changeInput('state', table.state());

    // search info
    var updateSearchInfo = function() {
      changeInput('search', table.search());
      if (filterRow) changeInput('search_columns', filterRow.toArray().map(function(td) {
        return $(td).find('input').first().val();
      }));
    }
    table.on('draw.dt', updateSearchInfo);
    updateSearchInfo();

    var cellInfo = function(thiz) {
      var info = tweakCellIndex(table.cell(thiz));
      info.value = table.cell(thiz).data();
      return info;
    }
    // the current cell clicked on
    table.on('click.dt', 'tbody td', function() {
      changeInput('cell_clicked', cellInfo(this));
    })
    changeInput('cell_clicked', {});

    // do not trigger table selection when clicking on links unless they have classes
    table.on('click.dt', 'tbody td a', function(e) {
      if (this.className === '') e.stopPropagation();
    });

    methods.addRow = function(data, rowname) {
      var data0 = table.row(0).data(), n = data0.length, d = n - data.length;
      if (d === 1) {
        data = rowname.concat(data)
      } else if (d !== 0) {
        console.log(data);
        console.log(data0);
        throw 'New data must be of the same length as current data (' + n + ')';
      };
      table.row.add(data).draw();
    }

    methods.updateSearch = function(keywords) {
      if (keywords.global !== null)
        $(table.table().container()).find('input[type=search]').first()
             .val(keywords.global).trigger('input');
      var columns = keywords.columns;
      if (!filterRow || columns === null) return;
      filterRow.toArray().map(function(td, i) {
        var v = typeof columns === 'string' ? columns : columns[i];
        if (typeof v === 'undefined') {
          console.log('The search keyword for column ' + i + ' is undefined')
          return;
        }
        $(td).find('input').first().val(v);
        searchColumn(i, v);
      });
      table.draw();
    }

    methods.hideCols = function(hide, reset) {
      if (reset) table.columns().visible(true, false);
      table.columns(hide).visible(false);
    }

    methods.showCols = function(show, reset) {
      if (reset) table.columns().visible(false, false);
      table.columns(show).visible(true);
    }

    methods.colReorder = function(order, origOrder) {
      table.colReorder.order(order, origOrder);
    }

    methods.selectPage = function(page) {
      if (table.page.info().pages < page || page < 1) {
        throw 'Selected page is out of range';
      };
      table.page(page - 1).draw(false);
    }

    methods.reloadData = function(resetPaging, clearSelection) {
      // empty selections first if necessary
      if (methods.selectRows && inArray('row', clearSelection)) methods.selectRows([]);
      if (methods.selectColumns && inArray('column', clearSelection)) methods.selectColumns([]);
      if (methods.selectCells && inArray('cell', clearSelection)) methods.selectCells([]);
      table.ajax.reload(null, resetPaging);
    }

    table.shinyMethods = methods;
  },
  resize: function(el, width, height, instance) {
    if (instance.data) this.renderValue(el, instance.data, instance);

    // dynamically adjust height if fillContainer = TRUE
    if (instance.fillContainer)
      this.fillAvailableHeight(el, height);

    this.adjustWidth(el);
  },

  // dynamically set the scroll body to fill available height
  // (used with fillContainer = TRUE)
  fillAvailableHeight: function(el, availableHeight) {

    // see how much of the table is occupied by header/footer elements
    // and use that to compute a target scroll body height
    var dtWrapper = $(el).find('div.dataTables_wrapper');
    var dtScrollBody = $(el).find($('div.dataTables_scrollBody'));
    var framingHeight = dtWrapper.innerHeight() - dtScrollBody.innerHeight();
    var scrollBodyHeight = availableHeight - framingHeight;

    // set the height
    dtScrollBody.height(scrollBodyHeight + 'px');
  },

  // adjust the width of columns; remove the hard-coded widths on table and the
  // scroll header when scrollX/Y are enabled
  adjustWidth: function(el) {
    var $el = $(el), table = $el.data('datatable');
    if (table) table.columns.adjust();
    $el.find('.dataTables_scrollHeadInner').css('width', '')
        .children('table').css('margin-left', '');
  }
});

  if (!HTMLWidgets.shinyMode) return;

  Shiny.addCustomMessageHandler('datatable-calls', function(data) {
    var id = data.id;
    var el = document.getElementById(id);
    var table = el ? $(el).data('datatable') : null;
    if (!table) {
      console.log("Couldn't find table with id " + id);
      return;
    }

    var methods = table.shinyMethods, call = data.call;
    if (methods[call.method]) {
      methods[call.method].apply(table, call.args);
    } else {
      console.log("Unknown method " + call.method);
    }
  });

})();
</script>
<style type="text/css">table.dataTable{width:100%;margin:0 auto;clear:both;border-collapse:separate;border-spacing:0}table.dataTable thead th,table.dataTable tfoot th{font-weight:bold}table.dataTable thead th,table.dataTable thead td{padding:10px 18px;border-bottom:1px solid #111}table.dataTable thead th:active,table.dataTable thead td:active{outline:none}table.dataTable tfoot th,table.dataTable tfoot td{padding:10px 18px 6px 18px;border-top:1px solid #111}table.dataTable thead .sorting,table.dataTable thead .sorting_asc,table.dataTable thead .sorting_desc,table.dataTable thead .sorting_asc_disabled,table.dataTable thead .sorting_desc_disabled{cursor:pointer;*cursor:hand;background-repeat:no-repeat;background-position:center right}table.dataTable thead .sorting{background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAQAAADYWf5HAAAAkElEQVQoz7XQMQ5AQBCF4dWQSJxC5wwax1Cq1e7BAdxD5SL+Tq/QCM1oNiJidwox0355mXnG/DrEtIQ6azioNZQxI0ykPhTQIwhCR+BmBYtlK7kLJYwWCcJA9M4qdrZrd8pPjZWPtOqdRQy320YSV17OatFC4euts6z39GYMKRPCTKY9UnPQ6P+GtMRfGtPnBCiqhAeJPmkqAAAAAElFTkSuQmCC)}table.dataTable thead .sorting_asc{background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAYAAAByUDbMAAAAZ0lEQVQ4y2NgGLKgquEuFxBPAGI2ahhWCsS/gDibUoO0gPgxEP8H4ttArEyuQYxAPBdqEAxPBImTY5gjEL9DM+wTENuQahAvEO9DMwiGdwAxOymGJQLxTyD+jgWDxCMZRsEoGAVoAADeemwtPcZI2wAAAABJRU5ErkJggg==)}table.dataTable thead .sorting_desc{background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAYAAAByUDbMAAAAZUlEQVQ4y2NgGAWjYBSggaqGu5FA/BOIv2PBIPFEUgxjB+IdQPwfC94HxLykus4GiD+hGfQOiB3J8SojEE9EM2wuSJzcsFMG4ttQgx4DsRalkZENxL+AuJQaMcsGxBOAmGvopk8AVz1sLZgg0bsAAAAASUVORK5CYII=)}table.dataTable thead .sorting_asc_disabled{background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAQAAADYWf5HAAAAW0lEQVQoz2NgoCm4w3Vnwh02wspK7/y6k01Ikdadx3f+37l9RxmfIsY7c4GKQHDiHUbcyhzvvIMq+3THBpci3jv7oIpAcMcdduzKEu/8vPMdDn/eiWQYBYMKAAC3ykIEuYQJUgAAAABJRU5ErkJggg==)}table.dataTable thead .sorting_desc_disabled{background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABMAAAATCAQAAADYWf5HAAAAWUlEQVQoz2NgGAWDCtyJvPPzznc4/HknEbsy9js77vyHw313eHGZZ3PnE1TRuzuOuK1lvDMRqmzuHUZ87lO+cxuo6PEdLUIeyb7z604pYf+y3Zlwh4u2YQoAc7ZCBHH4jigAAAAASUVORK5CYII=)}table.dataTable tbody tr{background-color:#ffffff}table.dataTable tbody tr.selected{background-color:#B0BED9}table.dataTable tbody th,table.dataTable tbody td{padding:8px 10px}table.dataTable.row-border tbody th,table.dataTable.row-border tbody td,table.dataTable.display tbody th,table.dataTable.display tbody td{border-top:1px solid #ddd}table.dataTable.row-border tbody tr:first-child th,table.dataTable.row-border tbody tr:first-child td,table.dataTable.display tbody tr:first-child th,table.dataTable.display tbody tr:first-child td{border-top:none}table.dataTable.cell-border tbody th,table.dataTable.cell-border tbody td{border-top:1px solid #ddd;border-right:1px solid #ddd}table.dataTable.cell-border tbody tr th:first-child,table.dataTable.cell-border tbody tr td:first-child{border-left:1px solid #ddd}table.dataTable.cell-border tbody tr:first-child th,table.dataTable.cell-border tbody tr:first-child td{border-top:none}table.dataTable.stripe tbody tr.odd,table.dataTable.display tbody tr.odd{background-color:#f9f9f9}table.dataTable.stripe tbody tr.odd.selected,table.dataTable.display tbody tr.odd.selected{background-color:#acbad4}table.dataTable.hover tbody tr:hover,table.dataTable.display tbody tr:hover{background-color:#f6f6f6}table.dataTable.hover tbody tr:hover.selected,table.dataTable.display tbody tr:hover.selected{background-color:#aab7d1}table.dataTable.order-column tbody tr>.sorting_1,table.dataTable.order-column tbody tr>.sorting_2,table.dataTable.order-column tbody tr>.sorting_3,table.dataTable.display tbody tr>.sorting_1,table.dataTable.display tbody tr>.sorting_2,table.dataTable.display tbody tr>.sorting_3{background-color:#fafafa}table.dataTable.order-column tbody tr.selected>.sorting_1,table.dataTable.order-column tbody tr.selected>.sorting_2,table.dataTable.order-column tbody tr.selected>.sorting_3,table.dataTable.display tbody tr.selected>.sorting_1,table.dataTable.display tbody tr.selected>.sorting_2,table.dataTable.display tbody tr.selected>.sorting_3{background-color:#acbad5}table.dataTable.display tbody tr.odd>.sorting_1,table.dataTable.order-column.stripe tbody tr.odd>.sorting_1{background-color:#f1f1f1}table.dataTable.display tbody tr.odd>.sorting_2,table.dataTable.order-column.stripe tbody tr.odd>.sorting_2{background-color:#f3f3f3}table.dataTable.display tbody tr.odd>.sorting_3,table.dataTable.order-column.stripe tbody tr.odd>.sorting_3{background-color:whitesmoke}table.dataTable.display tbody tr.odd.selected>.sorting_1,table.dataTable.order-column.stripe tbody tr.odd.selected>.sorting_1{background-color:#a6b4cd}table.dataTable.display tbody tr.odd.selected>.sorting_2,table.dataTable.order-column.stripe tbody tr.odd.selected>.sorting_2{background-color:#a8b5cf}table.dataTable.display tbody tr.odd.selected>.sorting_3,table.dataTable.order-column.stripe tbody tr.odd.selected>.sorting_3{background-color:#a9b7d1}table.dataTable.display tbody tr.even>.sorting_1,table.dataTable.order-column.stripe tbody tr.even>.sorting_1{background-color:#fafafa}table.dataTable.display tbody tr.even>.sorting_2,table.dataTable.order-column.stripe tbody tr.even>.sorting_2{background-color:#fcfcfc}table.dataTable.display tbody tr.even>.sorting_3,table.dataTable.order-column.stripe tbody tr.even>.sorting_3{background-color:#fefefe}table.dataTable.display tbody tr.even.selected>.sorting_1,table.dataTable.order-column.stripe tbody tr.even.selected>.sorting_1{background-color:#acbad5}table.dataTable.display tbody tr.even.selected>.sorting_2,table.dataTable.order-column.stripe tbody tr.even.selected>.sorting_2{background-color:#aebcd6}table.dataTable.display tbody tr.even.selected>.sorting_3,table.dataTable.order-column.stripe tbody tr.even.selected>.sorting_3{background-color:#afbdd8}table.dataTable.display tbody tr:hover>.sorting_1,table.dataTable.order-column.hover tbody tr:hover>.sorting_1{background-color:#eaeaea}table.dataTable.display tbody tr:hover>.sorting_2,table.dataTable.order-column.hover tbody tr:hover>.sorting_2{background-color:#ececec}table.dataTable.display tbody tr:hover>.sorting_3,table.dataTable.order-column.hover tbody tr:hover>.sorting_3{background-color:#efefef}table.dataTable.display tbody tr:hover.selected>.sorting_1,table.dataTable.order-column.hover tbody tr:hover.selected>.sorting_1{background-color:#a2aec7}table.dataTable.display tbody tr:hover.selected>.sorting_2,table.dataTable.order-column.hover tbody tr:hover.selected>.sorting_2{background-color:#a3b0c9}table.dataTable.display tbody tr:hover.selected>.sorting_3,table.dataTable.order-column.hover tbody tr:hover.selected>.sorting_3{background-color:#a5b2cb}table.dataTable.no-footer{border-bottom:1px solid #111}table.dataTable.nowrap th,table.dataTable.nowrap td{white-space:nowrap}table.dataTable.compact thead th,table.dataTable.compact thead td{padding:4px 17px 4px 4px}table.dataTable.compact tfoot th,table.dataTable.compact tfoot td{padding:4px}table.dataTable.compact tbody th,table.dataTable.compact tbody td{padding:4px}table.dataTable th.dt-left,table.dataTable td.dt-left{text-align:left}table.dataTable th.dt-center,table.dataTable td.dt-center,table.dataTable td.dataTables_empty{text-align:center}table.dataTable th.dt-right,table.dataTable td.dt-right{text-align:right}table.dataTable th.dt-justify,table.dataTable td.dt-justify{text-align:justify}table.dataTable th.dt-nowrap,table.dataTable td.dt-nowrap{white-space:nowrap}table.dataTable thead th.dt-head-left,table.dataTable thead td.dt-head-left,table.dataTable tfoot th.dt-head-left,table.dataTable tfoot td.dt-head-left{text-align:left}table.dataTable thead th.dt-head-center,table.dataTable thead td.dt-head-center,table.dataTable tfoot th.dt-head-center,table.dataTable tfoot td.dt-head-center{text-align:center}table.dataTable thead th.dt-head-right,table.dataTable thead td.dt-head-right,table.dataTable tfoot th.dt-head-right,table.dataTable tfoot td.dt-head-right{text-align:right}table.dataTable thead th.dt-head-justify,table.dataTable thead td.dt-head-justify,table.dataTable tfoot th.dt-head-justify,table.dataTable tfoot td.dt-head-justify{text-align:justify}table.dataTable thead th.dt-head-nowrap,table.dataTable thead td.dt-head-nowrap,table.dataTable tfoot th.dt-head-nowrap,table.dataTable tfoot td.dt-head-nowrap{white-space:nowrap}table.dataTable tbody th.dt-body-left,table.dataTable tbody td.dt-body-left{text-align:left}table.dataTable tbody th.dt-body-center,table.dataTable tbody td.dt-body-center{text-align:center}table.dataTable tbody th.dt-body-right,table.dataTable tbody td.dt-body-right{text-align:right}table.dataTable tbody th.dt-body-justify,table.dataTable tbody td.dt-body-justify{text-align:justify}table.dataTable tbody th.dt-body-nowrap,table.dataTable tbody td.dt-body-nowrap{white-space:nowrap}table.dataTable,table.dataTable th,table.dataTable td{box-sizing:content-box}.dataTables_wrapper{position:relative;clear:both;*zoom:1;zoom:1}.dataTables_wrapper .dataTables_length{float:left}.dataTables_wrapper .dataTables_filter{float:right;text-align:right}.dataTables_wrapper .dataTables_filter input{margin-left:0.5em}.dataTables_wrapper .dataTables_info{clear:both;float:left;padding-top:0.755em}.dataTables_wrapper .dataTables_paginate{float:right;text-align:right;padding-top:0.25em}.dataTables_wrapper .dataTables_paginate .paginate_button{box-sizing:border-box;display:inline-block;min-width:1.5em;padding:0.5em 1em;margin-left:2px;text-align:center;text-decoration:none !important;cursor:pointer;*cursor:hand;color:#333 !important;border:1px solid transparent;border-radius:2px}.dataTables_wrapper .dataTables_paginate .paginate_button.current,.dataTables_wrapper .dataTables_paginate .paginate_button.current:hover{color:#333 !important;border:1px solid #979797;background-color:white;background:-webkit-gradient(linear, left top, left bottom, color-stop(0%, #fff), color-stop(100%, #dcdcdc));background:-webkit-linear-gradient(top, #fff 0%, #dcdcdc 100%);background:-moz-linear-gradient(top, #fff 0%, #dcdcdc 100%);background:-ms-linear-gradient(top, #fff 0%, #dcdcdc 100%);background:-o-linear-gradient(top, #fff 0%, #dcdcdc 100%);background:linear-gradient(to bottom, #fff 0%, #dcdcdc 100%)}.dataTables_wrapper .dataTables_paginate .paginate_button.disabled,.dataTables_wrapper .dataTables_paginate .paginate_button.disabled:hover,.dataTables_wrapper .dataTables_paginate .paginate_button.disabled:active{cursor:default;color:#666 !important;border:1px solid transparent;background:transparent;box-shadow:none}.dataTables_wrapper .dataTables_paginate .paginate_button:hover{color:white !important;border:1px solid #111;background-color:#585858;background:-webkit-gradient(linear, left top, left bottom, color-stop(0%, #585858), color-stop(100%, #111));background:-webkit-linear-gradient(top, #585858 0%, #111 100%);background:-moz-linear-gradient(top, #585858 0%, #111 100%);background:-ms-linear-gradient(top, #585858 0%, #111 100%);background:-o-linear-gradient(top, #585858 0%, #111 100%);background:linear-gradient(to bottom, #585858 0%, #111 100%)}.dataTables_wrapper .dataTables_paginate .paginate_button:active{outline:none;background-color:#2b2b2b;background:-webkit-gradient(linear, left top, left bottom, color-stop(0%, #2b2b2b), color-stop(100%, #0c0c0c));background:-webkit-linear-gradient(top, #2b2b2b 0%, #0c0c0c 100%);background:-moz-linear-gradient(top, #2b2b2b 0%, #0c0c0c 100%);background:-ms-linear-gradient(top, #2b2b2b 0%, #0c0c0c 100%);background:-o-linear-gradient(top, #2b2b2b 0%, #0c0c0c 100%);background:linear-gradient(to bottom, #2b2b2b 0%, #0c0c0c 100%);box-shadow:inset 0 0 3px #111}.dataTables_wrapper .dataTables_paginate .ellipsis{padding:0 1em}.dataTables_wrapper .dataTables_processing{position:absolute;top:50%;left:50%;width:100%;height:40px;margin-left:-50%;margin-top:-25px;padding-top:20px;text-align:center;font-size:1.2em;background-color:white;background:-webkit-gradient(linear, left top, right top, color-stop(0%, rgba(255,255,255,0)), color-stop(25%, rgba(255,255,255,0.9)), color-stop(75%, rgba(255,255,255,0.9)), color-stop(100%, rgba(255,255,255,0)));background:-webkit-linear-gradient(left, rgba(255,255,255,0) 0%, rgba(255,255,255,0.9) 25%, rgba(255,255,255,0.9) 75%, rgba(255,255,255,0) 100%);background:-moz-linear-gradient(left, rgba(255,255,255,0) 0%, rgba(255,255,255,0.9) 25%, rgba(255,255,255,0.9) 75%, rgba(255,255,255,0) 100%);background:-ms-linear-gradient(left, rgba(255,255,255,0) 0%, rgba(255,255,255,0.9) 25%, rgba(255,255,255,0.9) 75%, rgba(255,255,255,0) 100%);background:-o-linear-gradient(left, rgba(255,255,255,0) 0%, rgba(255,255,255,0.9) 25%, rgba(255,255,255,0.9) 75%, rgba(255,255,255,0) 100%);background:linear-gradient(to right, rgba(255,255,255,0) 0%, rgba(255,255,255,0.9) 25%, rgba(255,255,255,0.9) 75%, rgba(255,255,255,0) 100%)}.dataTables_wrapper .dataTables_length,.dataTables_wrapper .dataTables_filter,.dataTables_wrapper .dataTables_info,.dataTables_wrapper .dataTables_processing,.dataTables_wrapper .dataTables_paginate{color:#333}.dataTables_wrapper .dataTables_scroll{clear:both}.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody{*margin-top:-1px;-webkit-overflow-scrolling:touch}.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>thead>tr>th,.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>thead>tr>td,.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>tbody>tr>th,.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>tbody>tr>td{vertical-align:middle}.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>thead>tr>th>div.dataTables_sizing,.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>thead>tr>td>div.dataTables_sizing,.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>tbody>tr>th>div.dataTables_sizing,.dataTables_wrapper .dataTables_scroll div.dataTables_scrollBody>table>tbody>tr>td>div.dataTables_sizing{height:0;overflow:hidden;margin:0 !important;padding:0 !important}.dataTables_wrapper.no-footer .dataTables_scrollBody{border-bottom:1px solid #111}.dataTables_wrapper.no-footer div.dataTables_scrollHead table.dataTable,.dataTables_wrapper.no-footer div.dataTables_scrollBody>table{border-bottom:none}.dataTables_wrapper:after{visibility:hidden;display:block;content:"";clear:both;height:0}@media screen and (max-width: 767px){.dataTables_wrapper .dataTables_info,.dataTables_wrapper .dataTables_paginate{float:none;text-align:center}.dataTables_wrapper .dataTables_paginate{margin-top:0.5em}}@media screen and (max-width: 640px){.dataTables_wrapper .dataTables_length,.dataTables_wrapper .dataTables_filter{float:none;text-align:center}.dataTables_wrapper .dataTables_filter{margin-top:0.5em}}
</style>
<style type="text/css">
table.dataTable tr.selected td, table.dataTable td.selected {
background-color: #b0bed9 !important;
}

.dataTables_scrollBody .dataTables_sizing {
visibility: hidden;
}
</style>
<script>/*!
 DataTables 1.10.16
 ©2008-2017 SpryMedia Ltd - datatables.net/license
*/
(function(h){"function"===typeof define&&define.amd?define(["jquery"],function(E){return h(E,window,document)}):"object"===typeof exports?module.exports=function(E,G){E||(E=window);G||(G="undefined"!==typeof window?require("jquery"):require("jquery")(E));return h(G,E,E.document)}:h(jQuery,window,document)})(function(h,E,G,k){function X(a){var b,c,d={};h.each(a,function(e){if((b=e.match(/^([^A-Z]+?)([A-Z])/))&&-1!=="a aa ai ao as b fn i m o s ".indexOf(b[1]+" "))c=e.replace(b[0],b[2].toLowerCase()),
d[c]=e,"o"===b[1]&&X(a[e])});a._hungarianMap=d}function I(a,b,c){a._hungarianMap||X(a);var d;h.each(b,function(e){d=a._hungarianMap[e];if(d!==k&&(c||b[d]===k))"o"===d.charAt(0)?(b[d]||(b[d]={}),h.extend(!0,b[d],b[e]),I(a[d],b[d],c)):b[d]=b[e]})}function Ca(a){var b=m.defaults.oLanguage,c=a.sZeroRecords;!a.sEmptyTable&&(c&&"No data available in table"===b.sEmptyTable)&&F(a,a,"sZeroRecords","sEmptyTable");!a.sLoadingRecords&&(c&&"Loading..."===b.sLoadingRecords)&&F(a,a,"sZeroRecords","sLoadingRecords");
a.sInfoThousands&&(a.sThousands=a.sInfoThousands);(a=a.sDecimal)&&cb(a)}function db(a){A(a,"ordering","bSort");A(a,"orderMulti","bSortMulti");A(a,"orderClasses","bSortClasses");A(a,"orderCellsTop","bSortCellsTop");A(a,"order","aaSorting");A(a,"orderFixed","aaSortingFixed");A(a,"paging","bPaginate");A(a,"pagingType","sPaginationType");A(a,"pageLength","iDisplayLength");A(a,"searching","bFilter");"boolean"===typeof a.sScrollX&&(a.sScrollX=a.sScrollX?"100%":"");"boolean"===typeof a.scrollX&&(a.scrollX=
a.scrollX?"100%":"");if(a=a.aoSearchCols)for(var b=0,c=a.length;b<c;b++)a[b]&&I(m.models.oSearch,a[b])}function eb(a){A(a,"orderable","bSortable");A(a,"orderData","aDataSort");A(a,"orderSequence","asSorting");A(a,"orderDataType","sortDataType");var b=a.aDataSort;"number"===typeof b&&!h.isArray(b)&&(a.aDataSort=[b])}function fb(a){if(!m.__browser){var b={};m.__browser=b;var c=h("<div/>").css({position:"fixed",top:0,left:-1*h(E).scrollLeft(),height:1,width:1,overflow:"hidden"}).append(h("<div/>").css({position:"absolute",
top:1,left:1,width:100,overflow:"scroll"}).append(h("<div/>").css({width:"100%",height:10}))).appendTo("body"),d=c.children(),e=d.children();b.barWidth=d[0].offsetWidth-d[0].clientWidth;b.bScrollOversize=100===e[0].offsetWidth&&100!==d[0].clientWidth;b.bScrollbarLeft=1!==Math.round(e.offset().left);b.bBounding=c[0].getBoundingClientRect().width?!0:!1;c.remove()}h.extend(a.oBrowser,m.__browser);a.oScroll.iBarWidth=m.__browser.barWidth}function gb(a,b,c,d,e,f){var g,j=!1;c!==k&&(g=c,j=!0);for(;d!==
e;)a.hasOwnProperty(d)&&(g=j?b(g,a[d],d,a):a[d],j=!0,d+=f);return g}function Da(a,b){var c=m.defaults.column,d=a.aoColumns.length,c=h.extend({},m.models.oColumn,c,{nTh:b?b:G.createElement("th"),sTitle:c.sTitle?c.sTitle:b?b.innerHTML:"",aDataSort:c.aDataSort?c.aDataSort:[d],mData:c.mData?c.mData:d,idx:d});a.aoColumns.push(c);c=a.aoPreSearchCols;c[d]=h.extend({},m.models.oSearch,c[d]);ja(a,d,h(b).data())}function ja(a,b,c){var b=a.aoColumns[b],d=a.oClasses,e=h(b.nTh);if(!b.sWidthOrig){b.sWidthOrig=
e.attr("width")||null;var f=(e.attr("style")||"").match(/width:\s*(\d+[pxem%]+)/);f&&(b.sWidthOrig=f[1])}c!==k&&null!==c&&(eb(c),I(m.defaults.column,c),c.mDataProp!==k&&!c.mData&&(c.mData=c.mDataProp),c.sType&&(b._sManualType=c.sType),c.className&&!c.sClass&&(c.sClass=c.className),c.sClass&&e.addClass(c.sClass),h.extend(b,c),F(b,c,"sWidth","sWidthOrig"),c.iDataSort!==k&&(b.aDataSort=[c.iDataSort]),F(b,c,"aDataSort"));var g=b.mData,j=Q(g),i=b.mRender?Q(b.mRender):null,c=function(a){return"string"===
typeof a&&-1!==a.indexOf("@")};b._bAttrSrc=h.isPlainObject(g)&&(c(g.sort)||c(g.type)||c(g.filter));b._setter=null;b.fnGetData=function(a,b,c){var d=j(a,b,k,c);return i&&b?i(d,b,a,c):d};b.fnSetData=function(a,b,c){return R(g)(a,b,c)};"number"!==typeof g&&(a._rowReadObject=!0);a.oFeatures.bSort||(b.bSortable=!1,e.addClass(d.sSortableNone));a=-1!==h.inArray("asc",b.asSorting);c=-1!==h.inArray("desc",b.asSorting);!b.bSortable||!a&&!c?(b.sSortingClass=d.sSortableNone,b.sSortingClassJUI=""):a&&!c?(b.sSortingClass=
d.sSortableAsc,b.sSortingClassJUI=d.sSortJUIAscAllowed):!a&&c?(b.sSortingClass=d.sSortableDesc,b.sSortingClassJUI=d.sSortJUIDescAllowed):(b.sSortingClass=d.sSortable,b.sSortingClassJUI=d.sSortJUI)}function Y(a){if(!1!==a.oFeatures.bAutoWidth){var b=a.aoColumns;Ea(a);for(var c=0,d=b.length;c<d;c++)b[c].nTh.style.width=b[c].sWidth}b=a.oScroll;(""!==b.sY||""!==b.sX)&&ka(a);r(a,null,"column-sizing",[a])}function Z(a,b){var c=la(a,"bVisible");return"number"===typeof c[b]?c[b]:null}function $(a,b){var c=
la(a,"bVisible"),c=h.inArray(b,c);return-1!==c?c:null}function aa(a){var b=0;h.each(a.aoColumns,function(a,d){d.bVisible&&"none"!==h(d.nTh).css("display")&&b++});return b}function la(a,b){var c=[];h.map(a.aoColumns,function(a,e){a[b]&&c.push(e)});return c}function Fa(a){var b=a.aoColumns,c=a.aoData,d=m.ext.type.detect,e,f,g,j,i,h,l,q,t;e=0;for(f=b.length;e<f;e++)if(l=b[e],t=[],!l.sType&&l._sManualType)l.sType=l._sManualType;else if(!l.sType){g=0;for(j=d.length;g<j;g++){i=0;for(h=c.length;i<h;i++){t[i]===
k&&(t[i]=B(a,i,e,"type"));q=d[g](t[i],a);if(!q&&g!==d.length-1)break;if("html"===q)break}if(q){l.sType=q;break}}l.sType||(l.sType="string")}}function hb(a,b,c,d){var e,f,g,j,i,n,l=a.aoColumns;if(b)for(e=b.length-1;0<=e;e--){n=b[e];var q=n.targets!==k?n.targets:n.aTargets;h.isArray(q)||(q=[q]);f=0;for(g=q.length;f<g;f++)if("number"===typeof q[f]&&0<=q[f]){for(;l.length<=q[f];)Da(a);d(q[f],n)}else if("number"===typeof q[f]&&0>q[f])d(l.length+q[f],n);else if("string"===typeof q[f]){j=0;for(i=l.length;j<
i;j++)("_all"==q[f]||h(l[j].nTh).hasClass(q[f]))&&d(j,n)}}if(c){e=0;for(a=c.length;e<a;e++)d(e,c[e])}}function M(a,b,c,d){var e=a.aoData.length,f=h.extend(!0,{},m.models.oRow,{src:c?"dom":"data",idx:e});f._aData=b;a.aoData.push(f);for(var g=a.aoColumns,j=0,i=g.length;j<i;j++)g[j].sType=null;a.aiDisplayMaster.push(e);b=a.rowIdFn(b);b!==k&&(a.aIds[b]=f);(c||!a.oFeatures.bDeferRender)&&Ga(a,e,c,d);return e}function ma(a,b){var c;b instanceof h||(b=h(b));return b.map(function(b,e){c=Ha(a,e);return M(a,
c.data,e,c.cells)})}function B(a,b,c,d){var e=a.iDraw,f=a.aoColumns[c],g=a.aoData[b]._aData,j=f.sDefaultContent,i=f.fnGetData(g,d,{settings:a,row:b,col:c});if(i===k)return a.iDrawError!=e&&null===j&&(J(a,0,"Requested unknown parameter "+("function"==typeof f.mData?"{function}":"'"+f.mData+"'")+" for row "+b+", column "+c,4),a.iDrawError=e),j;if((i===g||null===i)&&null!==j&&d!==k)i=j;else if("function"===typeof i)return i.call(g);return null===i&&"display"==d?"":i}function ib(a,b,c,d){a.aoColumns[c].fnSetData(a.aoData[b]._aData,
d,{settings:a,row:b,col:c})}function Ia(a){return h.map(a.match(/(\\.|[^\.])+/g)||[""],function(a){return a.replace(/\\\./g,".")})}function Q(a){if(h.isPlainObject(a)){var b={};h.each(a,function(a,c){c&&(b[a]=Q(c))});return function(a,c,f,g){var j=b[c]||b._;return j!==k?j(a,c,f,g):a}}if(null===a)return function(a){return a};if("function"===typeof a)return function(b,c,f,g){return a(b,c,f,g)};if("string"===typeof a&&(-1!==a.indexOf(".")||-1!==a.indexOf("[")||-1!==a.indexOf("("))){var c=function(a,
b,f){var g,j;if(""!==f){j=Ia(f);for(var i=0,n=j.length;i<n;i++){f=j[i].match(ba);g=j[i].match(U);if(f){j[i]=j[i].replace(ba,"");""!==j[i]&&(a=a[j[i]]);g=[];j.splice(0,i+1);j=j.join(".");if(h.isArray(a)){i=0;for(n=a.length;i<n;i++)g.push(c(a[i],b,j))}a=f[0].substring(1,f[0].length-1);a=""===a?g:g.join(a);break}else if(g){j[i]=j[i].replace(U,"");a=a[j[i]]();continue}if(null===a||a[j[i]]===k)return k;a=a[j[i]]}}return a};return function(b,e){return c(b,e,a)}}return function(b){return b[a]}}function R(a){if(h.isPlainObject(a))return R(a._);
if(null===a)return function(){};if("function"===typeof a)return function(b,d,e){a(b,"set",d,e)};if("string"===typeof a&&(-1!==a.indexOf(".")||-1!==a.indexOf("[")||-1!==a.indexOf("("))){var b=function(a,d,e){var e=Ia(e),f;f=e[e.length-1];for(var g,j,i=0,n=e.length-1;i<n;i++){g=e[i].match(ba);j=e[i].match(U);if(g){e[i]=e[i].replace(ba,"");a[e[i]]=[];f=e.slice();f.splice(0,i+1);g=f.join(".");if(h.isArray(d)){j=0;for(n=d.length;j<n;j++)f={},b(f,d[j],g),a[e[i]].push(f)}else a[e[i]]=d;return}j&&(e[i]=e[i].replace(U,
""),a=a[e[i]](d));if(null===a[e[i]]||a[e[i]]===k)a[e[i]]={};a=a[e[i]]}if(f.match(U))a[f.replace(U,"")](d);else a[f.replace(ba,"")]=d};return function(c,d){return b(c,d,a)}}return function(b,d){b[a]=d}}function Ja(a){return D(a.aoData,"_aData")}function na(a){a.aoData.length=0;a.aiDisplayMaster.length=0;a.aiDisplay.length=0;a.aIds={}}function oa(a,b,c){for(var d=-1,e=0,f=a.length;e<f;e++)a[e]==b?d=e:a[e]>b&&a[e]--; -1!=d&&c===k&&a.splice(d,1)}function ca(a,b,c,d){var e=a.aoData[b],f,g=function(c,d){for(;c.childNodes.length;)c.removeChild(c.firstChild);
c.innerHTML=B(a,b,d,"display")};if("dom"===c||(!c||"auto"===c)&&"dom"===e.src)e._aData=Ha(a,e,d,d===k?k:e._aData).data;else{var j=e.anCells;if(j)if(d!==k)g(j[d],d);else{c=0;for(f=j.length;c<f;c++)g(j[c],c)}}e._aSortData=null;e._aFilterData=null;g=a.aoColumns;if(d!==k)g[d].sType=null;else{c=0;for(f=g.length;c<f;c++)g[c].sType=null;Ka(a,e)}}function Ha(a,b,c,d){var e=[],f=b.firstChild,g,j,i=0,n,l=a.aoColumns,q=a._rowReadObject,d=d!==k?d:q?{}:[],t=function(a,b){if("string"===typeof a){var c=a.indexOf("@");
-1!==c&&(c=a.substring(c+1),R(a)(d,b.getAttribute(c)))}},m=function(a){if(c===k||c===i)j=l[i],n=h.trim(a.innerHTML),j&&j._bAttrSrc?(R(j.mData._)(d,n),t(j.mData.sort,a),t(j.mData.type,a),t(j.mData.filter,a)):q?(j._setter||(j._setter=R(j.mData)),j._setter(d,n)):d[i]=n;i++};if(f)for(;f;){g=f.nodeName.toUpperCase();if("TD"==g||"TH"==g)m(f),e.push(f);f=f.nextSibling}else{e=b.anCells;f=0;for(g=e.length;f<g;f++)m(e[f])}if(b=b.firstChild?b:b.nTr)(b=b.getAttribute("id"))&&R(a.rowId)(d,b);return{data:d,cells:e}}
function Ga(a,b,c,d){var e=a.aoData[b],f=e._aData,g=[],j,i,n,l,q;if(null===e.nTr){j=c||G.createElement("tr");e.nTr=j;e.anCells=g;j._DT_RowIndex=b;Ka(a,e);l=0;for(q=a.aoColumns.length;l<q;l++){n=a.aoColumns[l];i=c?d[l]:G.createElement(n.sCellType);i._DT_CellIndex={row:b,column:l};g.push(i);if((!c||n.mRender||n.mData!==l)&&(!h.isPlainObject(n.mData)||n.mData._!==l+".display"))i.innerHTML=B(a,b,l,"display");n.sClass&&(i.className+=" "+n.sClass);n.bVisible&&!c?j.appendChild(i):!n.bVisible&&c&&i.parentNode.removeChild(i);
n.fnCreatedCell&&n.fnCreatedCell.call(a.oInstance,i,B(a,b,l),f,b,l)}r(a,"aoRowCreatedCallback",null,[j,f,b])}e.nTr.setAttribute("role","row")}function Ka(a,b){var c=b.nTr,d=b._aData;if(c){var e=a.rowIdFn(d);e&&(c.id=e);d.DT_RowClass&&(e=d.DT_RowClass.split(" "),b.__rowc=b.__rowc?qa(b.__rowc.concat(e)):e,h(c).removeClass(b.__rowc.join(" ")).addClass(d.DT_RowClass));d.DT_RowAttr&&h(c).attr(d.DT_RowAttr);d.DT_RowData&&h(c).data(d.DT_RowData)}}function jb(a){var b,c,d,e,f,g=a.nTHead,j=a.nTFoot,i=0===
h("th, td",g).length,n=a.oClasses,l=a.aoColumns;i&&(e=h("<tr/>").appendTo(g));b=0;for(c=l.length;b<c;b++)f=l[b],d=h(f.nTh).addClass(f.sClass),i&&d.appendTo(e),a.oFeatures.bSort&&(d.addClass(f.sSortingClass),!1!==f.bSortable&&(d.attr("tabindex",a.iTabIndex).attr("aria-controls",a.sTableId),La(a,f.nTh,b))),f.sTitle!=d[0].innerHTML&&d.html(f.sTitle),Ma(a,"header")(a,d,f,n);i&&da(a.aoHeader,g);h(g).find(">tr").attr("role","row");h(g).find(">tr>th, >tr>td").addClass(n.sHeaderTH);h(j).find(">tr>th, >tr>td").addClass(n.sFooterTH);
if(null!==j){a=a.aoFooter[0];b=0;for(c=a.length;b<c;b++)f=l[b],f.nTf=a[b].cell,f.sClass&&h(f.nTf).addClass(f.sClass)}}function ea(a,b,c){var d,e,f,g=[],j=[],i=a.aoColumns.length,n;if(b){c===k&&(c=!1);d=0;for(e=b.length;d<e;d++){g[d]=b[d].slice();g[d].nTr=b[d].nTr;for(f=i-1;0<=f;f--)!a.aoColumns[f].bVisible&&!c&&g[d].splice(f,1);j.push([])}d=0;for(e=g.length;d<e;d++){if(a=g[d].nTr)for(;f=a.firstChild;)a.removeChild(f);f=0;for(b=g[d].length;f<b;f++)if(n=i=1,j[d][f]===k){a.appendChild(g[d][f].cell);
for(j[d][f]=1;g[d+i]!==k&&g[d][f].cell==g[d+i][f].cell;)j[d+i][f]=1,i++;for(;g[d][f+n]!==k&&g[d][f].cell==g[d][f+n].cell;){for(c=0;c<i;c++)j[d+c][f+n]=1;n++}h(g[d][f].cell).attr("rowspan",i).attr("colspan",n)}}}}function N(a){var b=r(a,"aoPreDrawCallback","preDraw",[a]);if(-1!==h.inArray(!1,b))C(a,!1);else{var b=[],c=0,d=a.asStripeClasses,e=d.length,f=a.oLanguage,g=a.iInitDisplayStart,j="ssp"==y(a),i=a.aiDisplay;a.bDrawing=!0;g!==k&&-1!==g&&(a._iDisplayStart=j?g:g>=a.fnRecordsDisplay()?0:g,a.iInitDisplayStart=
-1);var g=a._iDisplayStart,n=a.fnDisplayEnd();if(a.bDeferLoading)a.bDeferLoading=!1,a.iDraw++,C(a,!1);else if(j){if(!a.bDestroying&&!kb(a))return}else a.iDraw++;if(0!==i.length){f=j?a.aoData.length:n;for(j=j?0:g;j<f;j++){var l=i[j],q=a.aoData[l];null===q.nTr&&Ga(a,l);l=q.nTr;if(0!==e){var t=d[c%e];q._sRowStripe!=t&&(h(l).removeClass(q._sRowStripe).addClass(t),q._sRowStripe=t)}r(a,"aoRowCallback",null,[l,q._aData,c,j]);b.push(l);c++}}else c=f.sZeroRecords,1==a.iDraw&&"ajax"==y(a)?c=f.sLoadingRecords:
f.sEmptyTable&&0===a.fnRecordsTotal()&&(c=f.sEmptyTable),b[0]=h("<tr/>",{"class":e?d[0]:""}).append(h("<td />",{valign:"top",colSpan:aa(a),"class":a.oClasses.sRowEmpty}).html(c))[0];r(a,"aoHeaderCallback","header",[h(a.nTHead).children("tr")[0],Ja(a),g,n,i]);r(a,"aoFooterCallback","footer",[h(a.nTFoot).children("tr")[0],Ja(a),g,n,i]);d=h(a.nTBody);d.children().detach();d.append(h(b));r(a,"aoDrawCallback","draw",[a]);a.bSorted=!1;a.bFiltered=!1;a.bDrawing=!1}}function S(a,b){var c=a.oFeatures,d=c.bFilter;
c.bSort&&lb(a);d?fa(a,a.oPreviousSearch):a.aiDisplay=a.aiDisplayMaster.slice();!0!==b&&(a._iDisplayStart=0);a._drawHold=b;N(a);a._drawHold=!1}function mb(a){var b=a.oClasses,c=h(a.nTable),c=h("<div/>").insertBefore(c),d=a.oFeatures,e=h("<div/>",{id:a.sTableId+"_wrapper","class":b.sWrapper+(a.nTFoot?"":" "+b.sNoFooter)});a.nHolding=c[0];a.nTableWrapper=e[0];a.nTableReinsertBefore=a.nTable.nextSibling;for(var f=a.sDom.split(""),g,j,i,n,l,q,k=0;k<f.length;k++){g=null;j=f[k];if("<"==j){i=h("<div/>")[0];
n=f[k+1];if("'"==n||'"'==n){l="";for(q=2;f[k+q]!=n;)l+=f[k+q],q++;"H"==l?l=b.sJUIHeader:"F"==l&&(l=b.sJUIFooter);-1!=l.indexOf(".")?(n=l.split("."),i.id=n[0].substr(1,n[0].length-1),i.className=n[1]):"#"==l.charAt(0)?i.id=l.substr(1,l.length-1):i.className=l;k+=q}e.append(i);e=h(i)}else if(">"==j)e=e.parent();else if("l"==j&&d.bPaginate&&d.bLengthChange)g=nb(a);else if("f"==j&&d.bFilter)g=ob(a);else if("r"==j&&d.bProcessing)g=pb(a);else if("t"==j)g=qb(a);else if("i"==j&&d.bInfo)g=rb(a);else if("p"==
j&&d.bPaginate)g=sb(a);else if(0!==m.ext.feature.length){i=m.ext.feature;q=0;for(n=i.length;q<n;q++)if(j==i[q].cFeature){g=i[q].fnInit(a);break}}g&&(i=a.aanFeatures,i[j]||(i[j]=[]),i[j].push(g),e.append(g))}c.replaceWith(e);a.nHolding=null}function da(a,b){var c=h(b).children("tr"),d,e,f,g,j,i,n,l,q,k;a.splice(0,a.length);f=0;for(i=c.length;f<i;f++)a.push([]);f=0;for(i=c.length;f<i;f++){d=c[f];for(e=d.firstChild;e;){if("TD"==e.nodeName.toUpperCase()||"TH"==e.nodeName.toUpperCase()){l=1*e.getAttribute("colspan");
q=1*e.getAttribute("rowspan");l=!l||0===l||1===l?1:l;q=!q||0===q||1===q?1:q;g=0;for(j=a[f];j[g];)g++;n=g;k=1===l?!0:!1;for(j=0;j<l;j++)for(g=0;g<q;g++)a[f+g][n+j]={cell:e,unique:k},a[f+g].nTr=d}e=e.nextSibling}}}function ra(a,b,c){var d=[];c||(c=a.aoHeader,b&&(c=[],da(c,b)));for(var b=0,e=c.length;b<e;b++)for(var f=0,g=c[b].length;f<g;f++)if(c[b][f].unique&&(!d[f]||!a.bSortCellsTop))d[f]=c[b][f].cell;return d}function sa(a,b,c){r(a,"aoServerParams","serverParams",[b]);if(b&&h.isArray(b)){var d={},
e=/(.*?)\[\]$/;h.each(b,function(a,b){var c=b.name.match(e);c?(c=c[0],d[c]||(d[c]=[]),d[c].push(b.value)):d[b.name]=b.value});b=d}var f,g=a.ajax,j=a.oInstance,i=function(b){r(a,null,"xhr",[a,b,a.jqXHR]);c(b)};if(h.isPlainObject(g)&&g.data){f=g.data;var n=h.isFunction(f)?f(b,a):f,b=h.isFunction(f)&&n?n:h.extend(!0,b,n);delete g.data}n={data:b,success:function(b){var c=b.error||b.sError;c&&J(a,0,c);a.json=b;i(b)},dataType:"json",cache:!1,type:a.sServerMethod,error:function(b,c){var d=r(a,null,"xhr",
[a,null,a.jqXHR]);-1===h.inArray(!0,d)&&("parsererror"==c?J(a,0,"Invalid JSON response",1):4===b.readyState&&J(a,0,"Ajax error",7));C(a,!1)}};a.oAjaxData=b;r(a,null,"preXhr",[a,b]);a.fnServerData?a.fnServerData.call(j,a.sAjaxSource,h.map(b,function(a,b){return{name:b,value:a}}),i,a):a.sAjaxSource||"string"===typeof g?a.jqXHR=h.ajax(h.extend(n,{url:g||a.sAjaxSource})):h.isFunction(g)?a.jqXHR=g.call(j,b,i,a):(a.jqXHR=h.ajax(h.extend(n,g)),g.data=f)}function kb(a){return a.bAjaxDataGet?(a.iDraw++,C(a,
!0),sa(a,tb(a),function(b){ub(a,b)}),!1):!0}function tb(a){var b=a.aoColumns,c=b.length,d=a.oFeatures,e=a.oPreviousSearch,f=a.aoPreSearchCols,g,j=[],i,n,l,k=V(a);g=a._iDisplayStart;i=!1!==d.bPaginate?a._iDisplayLength:-1;var t=function(a,b){j.push({name:a,value:b})};t("sEcho",a.iDraw);t("iColumns",c);t("sColumns",D(b,"sName").join(","));t("iDisplayStart",g);t("iDisplayLength",i);var pa={draw:a.iDraw,columns:[],order:[],start:g,length:i,search:{value:e.sSearch,regex:e.bRegex}};for(g=0;g<c;g++)n=b[g],
l=f[g],i="function"==typeof n.mData?"function":n.mData,pa.columns.push({data:i,name:n.sName,searchable:n.bSearchable,orderable:n.bSortable,search:{value:l.sSearch,regex:l.bRegex}}),t("mDataProp_"+g,i),d.bFilter&&(t("sSearch_"+g,l.sSearch),t("bRegex_"+g,l.bRegex),t("bSearchable_"+g,n.bSearchable)),d.bSort&&t("bSortable_"+g,n.bSortable);d.bFilter&&(t("sSearch",e.sSearch),t("bRegex",e.bRegex));d.bSort&&(h.each(k,function(a,b){pa.order.push({column:b.col,dir:b.dir});t("iSortCol_"+a,b.col);t("sSortDir_"+
a,b.dir)}),t("iSortingCols",k.length));b=m.ext.legacy.ajax;return null===b?a.sAjaxSource?j:pa:b?j:pa}function ub(a,b){var c=ta(a,b),d=b.sEcho!==k?b.sEcho:b.draw,e=b.iTotalRecords!==k?b.iTotalRecords:b.recordsTotal,f=b.iTotalDisplayRecords!==k?b.iTotalDisplayRecords:b.recordsFiltered;if(d){if(1*d<a.iDraw)return;a.iDraw=1*d}na(a);a._iRecordsTotal=parseInt(e,10);a._iRecordsDisplay=parseInt(f,10);d=0;for(e=c.length;d<e;d++)M(a,c[d]);a.aiDisplay=a.aiDisplayMaster.slice();a.bAjaxDataGet=!1;N(a);a._bInitComplete||
ua(a,b);a.bAjaxDataGet=!0;C(a,!1)}function ta(a,b){var c=h.isPlainObject(a.ajax)&&a.ajax.dataSrc!==k?a.ajax.dataSrc:a.sAjaxDataProp;return"data"===c?b.aaData||b[c]:""!==c?Q(c)(b):b}function ob(a){var b=a.oClasses,c=a.sTableId,d=a.oLanguage,e=a.oPreviousSearch,f=a.aanFeatures,g='<input type="search" class="'+b.sFilterInput+'"/>',j=d.sSearch,j=j.match(/_INPUT_/)?j.replace("_INPUT_",g):j+g,b=h("<div/>",{id:!f.f?c+"_filter":null,"class":b.sFilter}).append(h("<label/>").append(j)),f=function(){var b=!this.value?
"":this.value;b!=e.sSearch&&(fa(a,{sSearch:b,bRegex:e.bRegex,bSmart:e.bSmart,bCaseInsensitive:e.bCaseInsensitive}),a._iDisplayStart=0,N(a))},g=null!==a.searchDelay?a.searchDelay:"ssp"===y(a)?400:0,i=h("input",b).val(e.sSearch).attr("placeholder",d.sSearchPlaceholder).on("keyup.DT search.DT input.DT paste.DT cut.DT",g?Na(f,g):f).on("keypress.DT",function(a){if(13==a.keyCode)return!1}).attr("aria-controls",c);h(a.nTable).on("search.dt.DT",function(b,c){if(a===c)try{i[0]!==G.activeElement&&i.val(e.sSearch)}catch(d){}});
return b[0]}function fa(a,b,c){var d=a.oPreviousSearch,e=a.aoPreSearchCols,f=function(a){d.sSearch=a.sSearch;d.bRegex=a.bRegex;d.bSmart=a.bSmart;d.bCaseInsensitive=a.bCaseInsensitive};Fa(a);if("ssp"!=y(a)){vb(a,b.sSearch,c,b.bEscapeRegex!==k?!b.bEscapeRegex:b.bRegex,b.bSmart,b.bCaseInsensitive);f(b);for(b=0;b<e.length;b++)wb(a,e[b].sSearch,b,e[b].bEscapeRegex!==k?!e[b].bEscapeRegex:e[b].bRegex,e[b].bSmart,e[b].bCaseInsensitive);xb(a)}else f(b);a.bFiltered=!0;r(a,null,"search",[a])}function xb(a){for(var b=
m.ext.search,c=a.aiDisplay,d,e,f=0,g=b.length;f<g;f++){for(var j=[],i=0,n=c.length;i<n;i++)e=c[i],d=a.aoData[e],b[f](a,d._aFilterData,e,d._aData,i)&&j.push(e);c.length=0;h.merge(c,j)}}function wb(a,b,c,d,e,f){if(""!==b){for(var g=[],j=a.aiDisplay,d=Oa(b,d,e,f),e=0;e<j.length;e++)b=a.aoData[j[e]]._aFilterData[c],d.test(b)&&g.push(j[e]);a.aiDisplay=g}}function vb(a,b,c,d,e,f){var d=Oa(b,d,e,f),f=a.oPreviousSearch.sSearch,g=a.aiDisplayMaster,j,e=[];0!==m.ext.search.length&&(c=!0);j=yb(a);if(0>=b.length)a.aiDisplay=
g.slice();else{if(j||c||f.length>b.length||0!==b.indexOf(f)||a.bSorted)a.aiDisplay=g.slice();b=a.aiDisplay;for(c=0;c<b.length;c++)d.test(a.aoData[b[c]]._sFilterRow)&&e.push(b[c]);a.aiDisplay=e}}function Oa(a,b,c,d){a=b?a:Pa(a);c&&(a="^(?=.*?"+h.map(a.match(/"[^"]+"|[^ ]+/g)||[""],function(a){if('"'===a.charAt(0))var b=a.match(/^"(.*)"$/),a=b?b[1]:a;return a.replace('"',"")}).join(")(?=.*?")+").*$");return RegExp(a,d?"i":"")}function yb(a){var b=a.aoColumns,c,d,e,f,g,j,i,h,l=m.ext.type.search;c=!1;
d=0;for(f=a.aoData.length;d<f;d++)if(h=a.aoData[d],!h._aFilterData){j=[];e=0;for(g=b.length;e<g;e++)c=b[e],c.bSearchable?(i=B(a,d,e,"filter"),l[c.sType]&&(i=l[c.sType](i)),null===i&&(i=""),"string"!==typeof i&&i.toString&&(i=i.toString())):i="",i.indexOf&&-1!==i.indexOf("&")&&(va.innerHTML=i,i=Wb?va.textContent:va.innerText),i.replace&&(i=i.replace(/[\r\n]/g,"")),j.push(i);h._aFilterData=j;h._sFilterRow=j.join("  ");c=!0}return c}function zb(a){return{search:a.sSearch,smart:a.bSmart,regex:a.bRegex,
caseInsensitive:a.bCaseInsensitive}}function Ab(a){return{sSearch:a.search,bSmart:a.smart,bRegex:a.regex,bCaseInsensitive:a.caseInsensitive}}function rb(a){var b=a.sTableId,c=a.aanFeatures.i,d=h("<div/>",{"class":a.oClasses.sInfo,id:!c?b+"_info":null});c||(a.aoDrawCallback.push({fn:Bb,sName:"information"}),d.attr("role","status").attr("aria-live","polite"),h(a.nTable).attr("aria-describedby",b+"_info"));return d[0]}function Bb(a){var b=a.aanFeatures.i;if(0!==b.length){var c=a.oLanguage,d=a._iDisplayStart+
1,e=a.fnDisplayEnd(),f=a.fnRecordsTotal(),g=a.fnRecordsDisplay(),j=g?c.sInfo:c.sInfoEmpty;g!==f&&(j+=" "+c.sInfoFiltered);j+=c.sInfoPostFix;j=Cb(a,j);c=c.fnInfoCallback;null!==c&&(j=c.call(a.oInstance,a,d,e,f,g,j));h(b).html(j)}}function Cb(a,b){var c=a.fnFormatNumber,d=a._iDisplayStart+1,e=a._iDisplayLength,f=a.fnRecordsDisplay(),g=-1===e;return b.replace(/_START_/g,c.call(a,d)).replace(/_END_/g,c.call(a,a.fnDisplayEnd())).replace(/_MAX_/g,c.call(a,a.fnRecordsTotal())).replace(/_TOTAL_/g,c.call(a,
f)).replace(/_PAGE_/g,c.call(a,g?1:Math.ceil(d/e))).replace(/_PAGES_/g,c.call(a,g?1:Math.ceil(f/e)))}function ga(a){var b,c,d=a.iInitDisplayStart,e=a.aoColumns,f;c=a.oFeatures;var g=a.bDeferLoading;if(a.bInitialised){mb(a);jb(a);ea(a,a.aoHeader);ea(a,a.aoFooter);C(a,!0);c.bAutoWidth&&Ea(a);b=0;for(c=e.length;b<c;b++)f=e[b],f.sWidth&&(f.nTh.style.width=v(f.sWidth));r(a,null,"preInit",[a]);S(a);e=y(a);if("ssp"!=e||g)"ajax"==e?sa(a,[],function(c){var f=ta(a,c);for(b=0;b<f.length;b++)M(a,f[b]);a.iInitDisplayStart=
d;S(a);C(a,!1);ua(a,c)},a):(C(a,!1),ua(a))}else setTimeout(function(){ga(a)},200)}function ua(a,b){a._bInitComplete=!0;(b||a.oInit.aaData)&&Y(a);r(a,null,"plugin-init",[a,b]);r(a,"aoInitComplete","init",[a,b])}function Qa(a,b){var c=parseInt(b,10);a._iDisplayLength=c;Ra(a);r(a,null,"length",[a,c])}function nb(a){for(var b=a.oClasses,c=a.sTableId,d=a.aLengthMenu,e=h.isArray(d[0]),f=e?d[0]:d,d=e?d[1]:d,e=h("<select/>",{name:c+"_length","aria-controls":c,"class":b.sLengthSelect}),g=0,j=f.length;g<j;g++)e[0][g]=
new Option("number"===typeof d[g]?a.fnFormatNumber(d[g]):d[g],f[g]);var i=h("<div><label/></div>").addClass(b.sLength);a.aanFeatures.l||(i[0].id=c+"_length");i.children().append(a.oLanguage.sLengthMenu.replace("_MENU_",e[0].outerHTML));h("select",i).val(a._iDisplayLength).on("change.DT",function(){Qa(a,h(this).val());N(a)});h(a.nTable).on("length.dt.DT",function(b,c,d){a===c&&h("select",i).val(d)});return i[0]}function sb(a){var b=a.sPaginationType,c=m.ext.pager[b],d="function"===typeof c,e=function(a){N(a)},
b=h("<div/>").addClass(a.oClasses.sPaging+b)[0],f=a.aanFeatures;d||c.fnInit(a,b,e);f.p||(b.id=a.sTableId+"_paginate",a.aoDrawCallback.push({fn:function(a){if(d){var b=a._iDisplayStart,i=a._iDisplayLength,h=a.fnRecordsDisplay(),l=-1===i,b=l?0:Math.ceil(b/i),i=l?1:Math.ceil(h/i),h=c(b,i),k,l=0;for(k=f.p.length;l<k;l++)Ma(a,"pageButton")(a,f.p[l],l,h,b,i)}else c.fnUpdate(a,e)},sName:"pagination"}));return b}function Sa(a,b,c){var d=a._iDisplayStart,e=a._iDisplayLength,f=a.fnRecordsDisplay();0===f||-1===
e?d=0:"number"===typeof b?(d=b*e,d>f&&(d=0)):"first"==b?d=0:"previous"==b?(d=0<=e?d-e:0,0>d&&(d=0)):"next"==b?d+e<f&&(d+=e):"last"==b?d=Math.floor((f-1)/e)*e:J(a,0,"Unknown paging action: "+b,5);b=a._iDisplayStart!==d;a._iDisplayStart=d;b&&(r(a,null,"page",[a]),c&&N(a));return b}function pb(a){return h("<div/>",{id:!a.aanFeatures.r?a.sTableId+"_processing":null,"class":a.oClasses.sProcessing}).html(a.oLanguage.sProcessing).insertBefore(a.nTable)[0]}function C(a,b){a.oFeatures.bProcessing&&h(a.aanFeatures.r).css("display",
b?"block":"none");r(a,null,"processing",[a,b])}function qb(a){var b=h(a.nTable);b.attr("role","grid");var c=a.oScroll;if(""===c.sX&&""===c.sY)return a.nTable;var d=c.sX,e=c.sY,f=a.oClasses,g=b.children("caption"),j=g.length?g[0]._captionSide:null,i=h(b[0].cloneNode(!1)),n=h(b[0].cloneNode(!1)),l=b.children("tfoot");l.length||(l=null);i=h("<div/>",{"class":f.sScrollWrapper}).append(h("<div/>",{"class":f.sScrollHead}).css({overflow:"hidden",position:"relative",border:0,width:d?!d?null:v(d):"100%"}).append(h("<div/>",
{"class":f.sScrollHeadInner}).css({"box-sizing":"content-box",width:c.sXInner||"100%"}).append(i.removeAttr("id").css("margin-left",0).append("top"===j?g:null).append(b.children("thead"))))).append(h("<div/>",{"class":f.sScrollBody}).css({position:"relative",overflow:"auto",width:!d?null:v(d)}).append(b));l&&i.append(h("<div/>",{"class":f.sScrollFoot}).css({overflow:"hidden",border:0,width:d?!d?null:v(d):"100%"}).append(h("<div/>",{"class":f.sScrollFootInner}).append(n.removeAttr("id").css("margin-left",
0).append("bottom"===j?g:null).append(b.children("tfoot")))));var b=i.children(),k=b[0],f=b[1],t=l?b[2]:null;if(d)h(f).on("scroll.DT",function(){var a=this.scrollLeft;k.scrollLeft=a;l&&(t.scrollLeft=a)});h(f).css(e&&c.bCollapse?"max-height":"height",e);a.nScrollHead=k;a.nScrollBody=f;a.nScrollFoot=t;a.aoDrawCallback.push({fn:ka,sName:"scrolling"});return i[0]}function ka(a){var b=a.oScroll,c=b.sX,d=b.sXInner,e=b.sY,b=b.iBarWidth,f=h(a.nScrollHead),g=f[0].style,j=f.children("div"),i=j[0].style,n=j.children("table"),
j=a.nScrollBody,l=h(j),q=j.style,t=h(a.nScrollFoot).children("div"),m=t.children("table"),o=h(a.nTHead),p=h(a.nTable),s=p[0],r=s.style,u=a.nTFoot?h(a.nTFoot):null,x=a.oBrowser,T=x.bScrollOversize,Xb=D(a.aoColumns,"nTh"),O,K,P,w,Ta=[],y=[],z=[],A=[],B,C=function(a){a=a.style;a.paddingTop="0";a.paddingBottom="0";a.borderTopWidth="0";a.borderBottomWidth="0";a.height=0};K=j.scrollHeight>j.clientHeight;if(a.scrollBarVis!==K&&a.scrollBarVis!==k)a.scrollBarVis=K,Y(a);else{a.scrollBarVis=K;p.children("thead, tfoot").remove();
u&&(P=u.clone().prependTo(p),O=u.find("tr"),P=P.find("tr"));w=o.clone().prependTo(p);o=o.find("tr");K=w.find("tr");w.find("th, td").removeAttr("tabindex");c||(q.width="100%",f[0].style.width="100%");h.each(ra(a,w),function(b,c){B=Z(a,b);c.style.width=a.aoColumns[B].sWidth});u&&H(function(a){a.style.width=""},P);f=p.outerWidth();if(""===c){r.width="100%";if(T&&(p.find("tbody").height()>j.offsetHeight||"scroll"==l.css("overflow-y")))r.width=v(p.outerWidth()-b);f=p.outerWidth()}else""!==d&&(r.width=
v(d),f=p.outerWidth());H(C,K);H(function(a){z.push(a.innerHTML);Ta.push(v(h(a).css("width")))},K);H(function(a,b){if(h.inArray(a,Xb)!==-1)a.style.width=Ta[b]},o);h(K).height(0);u&&(H(C,P),H(function(a){A.push(a.innerHTML);y.push(v(h(a).css("width")))},P),H(function(a,b){a.style.width=y[b]},O),h(P).height(0));H(function(a,b){a.innerHTML='<div class="dataTables_sizing" style="height:0;overflow:hidden;">'+z[b]+"</div>";a.style.width=Ta[b]},K);u&&H(function(a,b){a.innerHTML='<div class="dataTables_sizing" style="height:0;overflow:hidden;">'+
A[b]+"</div>";a.style.width=y[b]},P);if(p.outerWidth()<f){O=j.scrollHeight>j.offsetHeight||"scroll"==l.css("overflow-y")?f+b:f;if(T&&(j.scrollHeight>j.offsetHeight||"scroll"==l.css("overflow-y")))r.width=v(O-b);(""===c||""!==d)&&J(a,1,"Possible column misalignment",6)}else O="100%";q.width=v(O);g.width=v(O);u&&(a.nScrollFoot.style.width=v(O));!e&&T&&(q.height=v(s.offsetHeight+b));c=p.outerWidth();n[0].style.width=v(c);i.width=v(c);d=p.height()>j.clientHeight||"scroll"==l.css("overflow-y");e="padding"+
(x.bScrollbarLeft?"Left":"Right");i[e]=d?b+"px":"0px";u&&(m[0].style.width=v(c),t[0].style.width=v(c),t[0].style[e]=d?b+"px":"0px");p.children("colgroup").insertBefore(p.children("thead"));l.scroll();if((a.bSorted||a.bFiltered)&&!a._drawHold)j.scrollTop=0}}function H(a,b,c){for(var d=0,e=0,f=b.length,g,j;e<f;){g=b[e].firstChild;for(j=c?c[e].firstChild:null;g;)1===g.nodeType&&(c?a(g,j,d):a(g,d),d++),g=g.nextSibling,j=c?j.nextSibling:null;e++}}function Ea(a){var b=a.nTable,c=a.aoColumns,d=a.oScroll,
e=d.sY,f=d.sX,g=d.sXInner,j=c.length,i=la(a,"bVisible"),n=h("th",a.nTHead),l=b.getAttribute("width"),k=b.parentNode,t=!1,m,o,p=a.oBrowser,d=p.bScrollOversize;(m=b.style.width)&&-1!==m.indexOf("%")&&(l=m);for(m=0;m<i.length;m++)o=c[i[m]],null!==o.sWidth&&(o.sWidth=Db(o.sWidthOrig,k),t=!0);if(d||!t&&!f&&!e&&j==aa(a)&&j==n.length)for(m=0;m<j;m++)i=Z(a,m),null!==i&&(c[i].sWidth=v(n.eq(m).width()));else{j=h(b).clone().css("visibility","hidden").removeAttr("id");j.find("tbody tr").remove();var s=h("<tr/>").appendTo(j.find("tbody"));
j.find("thead, tfoot").remove();j.append(h(a.nTHead).clone()).append(h(a.nTFoot).clone());j.find("tfoot th, tfoot td").css("width","");n=ra(a,j.find("thead")[0]);for(m=0;m<i.length;m++)o=c[i[m]],n[m].style.width=null!==o.sWidthOrig&&""!==o.sWidthOrig?v(o.sWidthOrig):"",o.sWidthOrig&&f&&h(n[m]).append(h("<div/>").css({width:o.sWidthOrig,margin:0,padding:0,border:0,height:1}));if(a.aoData.length)for(m=0;m<i.length;m++)t=i[m],o=c[t],h(Eb(a,t)).clone(!1).append(o.sContentPadding).appendTo(s);h("[name]",
j).removeAttr("name");o=h("<div/>").css(f||e?{position:"absolute",top:0,left:0,height:1,right:0,overflow:"hidden"}:{}).append(j).appendTo(k);f&&g?j.width(g):f?(j.css("width","auto"),j.removeAttr("width"),j.width()<k.clientWidth&&l&&j.width(k.clientWidth)):e?j.width(k.clientWidth):l&&j.width(l);for(m=e=0;m<i.length;m++)k=h(n[m]),g=k.outerWidth()-k.width(),k=p.bBounding?Math.ceil(n[m].getBoundingClientRect().width):k.outerWidth(),e+=k,c[i[m]].sWidth=v(k-g);b.style.width=v(e);o.remove()}l&&(b.style.width=
v(l));if((l||f)&&!a._reszEvt)b=function(){h(E).on("resize.DT-"+a.sInstance,Na(function(){Y(a)}))},d?setTimeout(b,1E3):b(),a._reszEvt=!0}function Db(a,b){if(!a)return 0;var c=h("<div/>").css("width",v(a)).appendTo(b||G.body),d=c[0].offsetWidth;c.remove();return d}function Eb(a,b){var c=Fb(a,b);if(0>c)return null;var d=a.aoData[c];return!d.nTr?h("<td/>").html(B(a,c,b,"display"))[0]:d.anCells[b]}function Fb(a,b){for(var c,d=-1,e=-1,f=0,g=a.aoData.length;f<g;f++)c=B(a,f,b,"display")+"",c=c.replace(Yb,
""),c=c.replace(/&nbsp;/g," "),c.length>d&&(d=c.length,e=f);return e}function v(a){return null===a?"0px":"number"==typeof a?0>a?"0px":a+"px":a.match(/\d$/)?a+"px":a}function V(a){var b,c,d=[],e=a.aoColumns,f,g,j,i;b=a.aaSortingFixed;c=h.isPlainObject(b);var n=[];f=function(a){a.length&&!h.isArray(a[0])?n.push(a):h.merge(n,a)};h.isArray(b)&&f(b);c&&b.pre&&f(b.pre);f(a.aaSorting);c&&b.post&&f(b.post);for(a=0;a<n.length;a++){i=n[a][0];f=e[i].aDataSort;b=0;for(c=f.length;b<c;b++)g=f[b],j=e[g].sType||
"string",n[a]._idx===k&&(n[a]._idx=h.inArray(n[a][1],e[g].asSorting)),d.push({src:i,col:g,dir:n[a][1],index:n[a]._idx,type:j,formatter:m.ext.type.order[j+"-pre"]})}return d}function lb(a){var b,c,d=[],e=m.ext.type.order,f=a.aoData,g=0,j,i=a.aiDisplayMaster,h;Fa(a);h=V(a);b=0;for(c=h.length;b<c;b++)j=h[b],j.formatter&&g++,Gb(a,j.col);if("ssp"!=y(a)&&0!==h.length){b=0;for(c=i.length;b<c;b++)d[i[b]]=b;g===h.length?i.sort(function(a,b){var c,e,g,j,i=h.length,k=f[a]._aSortData,m=f[b]._aSortData;for(g=
0;g<i;g++)if(j=h[g],c=k[j.col],e=m[j.col],c=c<e?-1:c>e?1:0,0!==c)return"asc"===j.dir?c:-c;c=d[a];e=d[b];return c<e?-1:c>e?1:0}):i.sort(function(a,b){var c,g,j,i,k=h.length,m=f[a]._aSortData,o=f[b]._aSortData;for(j=0;j<k;j++)if(i=h[j],c=m[i.col],g=o[i.col],i=e[i.type+"-"+i.dir]||e["string-"+i.dir],c=i(c,g),0!==c)return c;c=d[a];g=d[b];return c<g?-1:c>g?1:0})}a.bSorted=!0}function Hb(a){for(var b,c,d=a.aoColumns,e=V(a),a=a.oLanguage.oAria,f=0,g=d.length;f<g;f++){c=d[f];var j=c.asSorting;b=c.sTitle.replace(/<.*?>/g,
"");var i=c.nTh;i.removeAttribute("aria-sort");c.bSortable&&(0<e.length&&e[0].col==f?(i.setAttribute("aria-sort","asc"==e[0].dir?"ascending":"descending"),c=j[e[0].index+1]||j[0]):c=j[0],b+="asc"===c?a.sSortAscending:a.sSortDescending);i.setAttribute("aria-label",b)}}function Ua(a,b,c,d){var e=a.aaSorting,f=a.aoColumns[b].asSorting,g=function(a,b){var c=a._idx;c===k&&(c=h.inArray(a[1],f));return c+1<f.length?c+1:b?null:0};"number"===typeof e[0]&&(e=a.aaSorting=[e]);c&&a.oFeatures.bSortMulti?(c=h.inArray(b,
D(e,"0")),-1!==c?(b=g(e[c],!0),null===b&&1===e.length&&(b=0),null===b?e.splice(c,1):(e[c][1]=f[b],e[c]._idx=b)):(e.push([b,f[0],0]),e[e.length-1]._idx=0)):e.length&&e[0][0]==b?(b=g(e[0]),e.length=1,e[0][1]=f[b],e[0]._idx=b):(e.length=0,e.push([b,f[0]]),e[0]._idx=0);S(a);"function"==typeof d&&d(a)}function La(a,b,c,d){var e=a.aoColumns[c];Va(b,{},function(b){!1!==e.bSortable&&(a.oFeatures.bProcessing?(C(a,!0),setTimeout(function(){Ua(a,c,b.shiftKey,d);"ssp"!==y(a)&&C(a,!1)},0)):Ua(a,c,b.shiftKey,d))})}
function wa(a){var b=a.aLastSort,c=a.oClasses.sSortColumn,d=V(a),e=a.oFeatures,f,g;if(e.bSort&&e.bSortClasses){e=0;for(f=b.length;e<f;e++)g=b[e].src,h(D(a.aoData,"anCells",g)).removeClass(c+(2>e?e+1:3));e=0;for(f=d.length;e<f;e++)g=d[e].src,h(D(a.aoData,"anCells",g)).addClass(c+(2>e?e+1:3))}a.aLastSort=d}function Gb(a,b){var c=a.aoColumns[b],d=m.ext.order[c.sSortDataType],e;d&&(e=d.call(a.oInstance,a,b,$(a,b)));for(var f,g=m.ext.type.order[c.sType+"-pre"],j=0,i=a.aoData.length;j<i;j++)if(c=a.aoData[j],
c._aSortData||(c._aSortData=[]),!c._aSortData[b]||d)f=d?e[j]:B(a,j,b,"sort"),c._aSortData[b]=g?g(f):f}function xa(a){if(a.oFeatures.bStateSave&&!a.bDestroying){var b={time:+new Date,start:a._iDisplayStart,length:a._iDisplayLength,order:h.extend(!0,[],a.aaSorting),search:zb(a.oPreviousSearch),columns:h.map(a.aoColumns,function(b,d){return{visible:b.bVisible,search:zb(a.aoPreSearchCols[d])}})};r(a,"aoStateSaveParams","stateSaveParams",[a,b]);a.oSavedState=b;a.fnStateSaveCallback.call(a.oInstance,a,
b)}}function Ib(a,b,c){var d,e,f=a.aoColumns,b=function(b){if(b&&b.time){var g=r(a,"aoStateLoadParams","stateLoadParams",[a,b]);if(-1===h.inArray(!1,g)&&(g=a.iStateDuration,!(0<g&&b.time<+new Date-1E3*g)&&!(b.columns&&f.length!==b.columns.length))){a.oLoadedState=h.extend(!0,{},b);b.start!==k&&(a._iDisplayStart=b.start,a.iInitDisplayStart=b.start);b.length!==k&&(a._iDisplayLength=b.length);b.order!==k&&(a.aaSorting=[],h.each(b.order,function(b,c){a.aaSorting.push(c[0]>=f.length?[0,c[1]]:c)}));b.search!==
k&&h.extend(a.oPreviousSearch,Ab(b.search));if(b.columns){d=0;for(e=b.columns.length;d<e;d++)g=b.columns[d],g.visible!==k&&(f[d].bVisible=g.visible),g.search!==k&&h.extend(a.aoPreSearchCols[d],Ab(g.search))}r(a,"aoStateLoaded","stateLoaded",[a,b])}}c()};if(a.oFeatures.bStateSave){var g=a.fnStateLoadCallback.call(a.oInstance,a,b);g!==k&&b(g)}else c()}function ya(a){var b=m.settings,a=h.inArray(a,D(b,"nTable"));return-1!==a?b[a]:null}function J(a,b,c,d){c="DataTables warning: "+(a?"table id="+a.sTableId+
" - ":"")+c;d&&(c+=". For more information about this error, please see http://datatables.net/tn/"+d);if(b)E.console&&console.log&&console.log(c);else if(b=m.ext,b=b.sErrMode||b.errMode,a&&r(a,null,"error",[a,d,c]),"alert"==b)alert(c);else{if("throw"==b)throw Error(c);"function"==typeof b&&b(a,d,c)}}function F(a,b,c,d){h.isArray(c)?h.each(c,function(c,d){h.isArray(d)?F(a,b,d[0],d[1]):F(a,b,d)}):(d===k&&(d=c),b[c]!==k&&(a[d]=b[c]))}function Jb(a,b,c){var d,e;for(e in b)b.hasOwnProperty(e)&&(d=b[e],
h.isPlainObject(d)?(h.isPlainObject(a[e])||(a[e]={}),h.extend(!0,a[e],d)):a[e]=c&&"data"!==e&&"aaData"!==e&&h.isArray(d)?d.slice():d);return a}function Va(a,b,c){h(a).on("click.DT",b,function(b){a.blur();c(b)}).on("keypress.DT",b,function(a){13===a.which&&(a.preventDefault(),c(a))}).on("selectstart.DT",function(){return!1})}function z(a,b,c,d){c&&a[b].push({fn:c,sName:d})}function r(a,b,c,d){var e=[];b&&(e=h.map(a[b].slice().reverse(),function(b){return b.fn.apply(a.oInstance,d)}));null!==c&&(b=h.Event(c+
".dt"),h(a.nTable).trigger(b,d),e.push(b.result));return e}function Ra(a){var b=a._iDisplayStart,c=a.fnDisplayEnd(),d=a._iDisplayLength;b>=c&&(b=c-d);b-=b%d;if(-1===d||0>b)b=0;a._iDisplayStart=b}function Ma(a,b){var c=a.renderer,d=m.ext.renderer[b];return h.isPlainObject(c)&&c[b]?d[c[b]]||d._:"string"===typeof c?d[c]||d._:d._}function y(a){return a.oFeatures.bServerSide?"ssp":a.ajax||a.sAjaxSource?"ajax":"dom"}function ha(a,b){var c=[],c=Kb.numbers_length,d=Math.floor(c/2);b<=c?c=W(0,b):a<=d?(c=W(0,
c-2),c.push("ellipsis"),c.push(b-1)):(a>=b-1-d?c=W(b-(c-2),b):(c=W(a-d+2,a+d-1),c.push("ellipsis"),c.push(b-1)),c.splice(0,0,"ellipsis"),c.splice(0,0,0));c.DT_el="span";return c}function cb(a){h.each({num:function(b){return za(b,a)},"num-fmt":function(b){return za(b,a,Wa)},"html-num":function(b){return za(b,a,Aa)},"html-num-fmt":function(b){return za(b,a,Aa,Wa)}},function(b,c){x.type.order[b+a+"-pre"]=c;b.match(/^html\-/)&&(x.type.search[b+a]=x.type.search.html)})}function Lb(a){return function(){var b=
[ya(this[m.ext.iApiIndex])].concat(Array.prototype.slice.call(arguments));return m.ext.internal[a].apply(this,b)}}var m=function(a){this.$=function(a,b){return this.api(!0).$(a,b)};this._=function(a,b){return this.api(!0).rows(a,b).data()};this.api=function(a){return a?new s(ya(this[x.iApiIndex])):new s(this)};this.fnAddData=function(a,b){var c=this.api(!0),d=h.isArray(a)&&(h.isArray(a[0])||h.isPlainObject(a[0]))?c.rows.add(a):c.row.add(a);(b===k||b)&&c.draw();return d.flatten().toArray()};this.fnAdjustColumnSizing=
function(a){var b=this.api(!0).columns.adjust(),c=b.settings()[0],d=c.oScroll;a===k||a?b.draw(!1):(""!==d.sX||""!==d.sY)&&ka(c)};this.fnClearTable=function(a){var b=this.api(!0).clear();(a===k||a)&&b.draw()};this.fnClose=function(a){this.api(!0).row(a).child.hide()};this.fnDeleteRow=function(a,b,c){var d=this.api(!0),a=d.rows(a),e=a.settings()[0],h=e.aoData[a[0][0]];a.remove();b&&b.call(this,e,h);(c===k||c)&&d.draw();return h};this.fnDestroy=function(a){this.api(!0).destroy(a)};this.fnDraw=function(a){this.api(!0).draw(a)};
this.fnFilter=function(a,b,c,d,e,h){e=this.api(!0);null===b||b===k?e.search(a,c,d,h):e.column(b).search(a,c,d,h);e.draw()};this.fnGetData=function(a,b){var c=this.api(!0);if(a!==k){var d=a.nodeName?a.nodeName.toLowerCase():"";return b!==k||"td"==d||"th"==d?c.cell(a,b).data():c.row(a).data()||null}return c.data().toArray()};this.fnGetNodes=function(a){var b=this.api(!0);return a!==k?b.row(a).node():b.rows().nodes().flatten().toArray()};this.fnGetPosition=function(a){var b=this.api(!0),c=a.nodeName.toUpperCase();
return"TR"==c?b.row(a).index():"TD"==c||"TH"==c?(a=b.cell(a).index(),[a.row,a.columnVisible,a.column]):null};this.fnIsOpen=function(a){return this.api(!0).row(a).child.isShown()};this.fnOpen=function(a,b,c){return this.api(!0).row(a).child(b,c).show().child()[0]};this.fnPageChange=function(a,b){var c=this.api(!0).page(a);(b===k||b)&&c.draw(!1)};this.fnSetColumnVis=function(a,b,c){a=this.api(!0).column(a).visible(b);(c===k||c)&&a.columns.adjust().draw()};this.fnSettings=function(){return ya(this[x.iApiIndex])};
this.fnSort=function(a){this.api(!0).order(a).draw()};this.fnSortListener=function(a,b,c){this.api(!0).order.listener(a,b,c)};this.fnUpdate=function(a,b,c,d,e){var h=this.api(!0);c===k||null===c?h.row(b).data(a):h.cell(b,c).data(a);(e===k||e)&&h.columns.adjust();(d===k||d)&&h.draw();return 0};this.fnVersionCheck=x.fnVersionCheck;var b=this,c=a===k,d=this.length;c&&(a={});this.oApi=this.internal=x.internal;for(var e in m.ext.internal)e&&(this[e]=Lb(e));this.each(function(){var e={},g=1<d?Jb(e,a,!0):
a,j=0,i,e=this.getAttribute("id"),n=!1,l=m.defaults,q=h(this);if("table"!=this.nodeName.toLowerCase())J(null,0,"Non-table node initialisation ("+this.nodeName+")",2);else{db(l);eb(l.column);I(l,l,!0);I(l.column,l.column,!0);I(l,h.extend(g,q.data()));var t=m.settings,j=0;for(i=t.length;j<i;j++){var o=t[j];if(o.nTable==this||o.nTHead.parentNode==this||o.nTFoot&&o.nTFoot.parentNode==this){var s=g.bRetrieve!==k?g.bRetrieve:l.bRetrieve;if(c||s)return o.oInstance;if(g.bDestroy!==k?g.bDestroy:l.bDestroy){o.oInstance.fnDestroy();
break}else{J(o,0,"Cannot reinitialise DataTable",3);return}}if(o.sTableId==this.id){t.splice(j,1);break}}if(null===e||""===e)this.id=e="DataTables_Table_"+m.ext._unique++;var p=h.extend(!0,{},m.models.oSettings,{sDestroyWidth:q[0].style.width,sInstance:e,sTableId:e});p.nTable=this;p.oApi=b.internal;p.oInit=g;t.push(p);p.oInstance=1===b.length?b:q.dataTable();db(g);g.oLanguage&&Ca(g.oLanguage);g.aLengthMenu&&!g.iDisplayLength&&(g.iDisplayLength=h.isArray(g.aLengthMenu[0])?g.aLengthMenu[0][0]:g.aLengthMenu[0]);
g=Jb(h.extend(!0,{},l),g);F(p.oFeatures,g,"bPaginate bLengthChange bFilter bSort bSortMulti bInfo bProcessing bAutoWidth bSortClasses bServerSide bDeferRender".split(" "));F(p,g,["asStripeClasses","ajax","fnServerData","fnFormatNumber","sServerMethod","aaSorting","aaSortingFixed","aLengthMenu","sPaginationType","sAjaxSource","sAjaxDataProp","iStateDuration","sDom","bSortCellsTop","iTabIndex","fnStateLoadCallback","fnStateSaveCallback","renderer","searchDelay","rowId",["iCookieDuration","iStateDuration"],
["oSearch","oPreviousSearch"],["aoSearchCols","aoPreSearchCols"],["iDisplayLength","_iDisplayLength"]]);F(p.oScroll,g,[["sScrollX","sX"],["sScrollXInner","sXInner"],["sScrollY","sY"],["bScrollCollapse","bCollapse"]]);F(p.oLanguage,g,"fnInfoCallback");z(p,"aoDrawCallback",g.fnDrawCallback,"user");z(p,"aoServerParams",g.fnServerParams,"user");z(p,"aoStateSaveParams",g.fnStateSaveParams,"user");z(p,"aoStateLoadParams",g.fnStateLoadParams,"user");z(p,"aoStateLoaded",g.fnStateLoaded,"user");z(p,"aoRowCallback",
g.fnRowCallback,"user");z(p,"aoRowCreatedCallback",g.fnCreatedRow,"user");z(p,"aoHeaderCallback",g.fnHeaderCallback,"user");z(p,"aoFooterCallback",g.fnFooterCallback,"user");z(p,"aoInitComplete",g.fnInitComplete,"user");z(p,"aoPreDrawCallback",g.fnPreDrawCallback,"user");p.rowIdFn=Q(g.rowId);fb(p);var u=p.oClasses;h.extend(u,m.ext.classes,g.oClasses);q.addClass(u.sTable);p.iInitDisplayStart===k&&(p.iInitDisplayStart=g.iDisplayStart,p._iDisplayStart=g.iDisplayStart);null!==g.iDeferLoading&&(p.bDeferLoading=
!0,e=h.isArray(g.iDeferLoading),p._iRecordsDisplay=e?g.iDeferLoading[0]:g.iDeferLoading,p._iRecordsTotal=e?g.iDeferLoading[1]:g.iDeferLoading);var v=p.oLanguage;h.extend(!0,v,g.oLanguage);v.sUrl&&(h.ajax({dataType:"json",url:v.sUrl,success:function(a){Ca(a);I(l.oLanguage,a);h.extend(true,v,a);ga(p)},error:function(){ga(p)}}),n=!0);null===g.asStripeClasses&&(p.asStripeClasses=[u.sStripeOdd,u.sStripeEven]);var e=p.asStripeClasses,x=q.children("tbody").find("tr").eq(0);-1!==h.inArray(!0,h.map(e,function(a){return x.hasClass(a)}))&&
(h("tbody tr",this).removeClass(e.join(" ")),p.asDestroyStripes=e.slice());e=[];t=this.getElementsByTagName("thead");0!==t.length&&(da(p.aoHeader,t[0]),e=ra(p));if(null===g.aoColumns){t=[];j=0;for(i=e.length;j<i;j++)t.push(null)}else t=g.aoColumns;j=0;for(i=t.length;j<i;j++)Da(p,e?e[j]:null);hb(p,g.aoColumnDefs,t,function(a,b){ja(p,a,b)});if(x.length){var w=function(a,b){return a.getAttribute("data-"+b)!==null?b:null};h(x[0]).children("th, td").each(function(a,b){var c=p.aoColumns[a];if(c.mData===
a){var d=w(b,"sort")||w(b,"order"),e=w(b,"filter")||w(b,"search");if(d!==null||e!==null){c.mData={_:a+".display",sort:d!==null?a+".@data-"+d:k,type:d!==null?a+".@data-"+d:k,filter:e!==null?a+".@data-"+e:k};ja(p,a)}}})}var T=p.oFeatures,e=function(){if(g.aaSorting===k){var a=p.aaSorting;j=0;for(i=a.length;j<i;j++)a[j][1]=p.aoColumns[j].asSorting[0]}wa(p);T.bSort&&z(p,"aoDrawCallback",function(){if(p.bSorted){var a=V(p),b={};h.each(a,function(a,c){b[c.src]=c.dir});r(p,null,"order",[p,a,b]);Hb(p)}});
z(p,"aoDrawCallback",function(){(p.bSorted||y(p)==="ssp"||T.bDeferRender)&&wa(p)},"sc");var a=q.children("caption").each(function(){this._captionSide=h(this).css("caption-side")}),b=q.children("thead");b.length===0&&(b=h("<thead/>").appendTo(q));p.nTHead=b[0];b=q.children("tbody");b.length===0&&(b=h("<tbody/>").appendTo(q));p.nTBody=b[0];b=q.children("tfoot");if(b.length===0&&a.length>0&&(p.oScroll.sX!==""||p.oScroll.sY!==""))b=h("<tfoot/>").appendTo(q);if(b.length===0||b.children().length===0)q.addClass(u.sNoFooter);
else if(b.length>0){p.nTFoot=b[0];da(p.aoFooter,p.nTFoot)}if(g.aaData)for(j=0;j<g.aaData.length;j++)M(p,g.aaData[j]);else(p.bDeferLoading||y(p)=="dom")&&ma(p,h(p.nTBody).children("tr"));p.aiDisplay=p.aiDisplayMaster.slice();p.bInitialised=true;n===false&&ga(p)};g.bStateSave?(T.bStateSave=!0,z(p,"aoDrawCallback",xa,"state_save"),Ib(p,g,e)):e()}});b=null;return this},x,s,o,u,Xa={},Mb=/[\r\n]/g,Aa=/<.*?>/g,Zb=/^\d{2,4}[\.\/\-]\d{1,2}[\.\/\-]\d{1,2}([T ]{1}\d{1,2}[:\.]\d{2}([\.:]\d{2})?)?$/,$b=RegExp("(\\/|\\.|\\*|\\+|\\?|\\||\\(|\\)|\\[|\\]|\\{|\\}|\\\\|\\$|\\^|\\-)",
"g"),Wa=/[',$£€¥%\u2009\u202F\u20BD\u20a9\u20BArfk]/gi,L=function(a){return!a||!0===a||"-"===a?!0:!1},Nb=function(a){var b=parseInt(a,10);return!isNaN(b)&&isFinite(a)?b:null},Ob=function(a,b){Xa[b]||(Xa[b]=RegExp(Pa(b),"g"));return"string"===typeof a&&"."!==b?a.replace(/\./g,"").replace(Xa[b],"."):a},Ya=function(a,b,c){var d="string"===typeof a;if(L(a))return!0;b&&d&&(a=Ob(a,b));c&&d&&(a=a.replace(Wa,""));return!isNaN(parseFloat(a))&&isFinite(a)},Pb=function(a,b,c){return L(a)?!0:!(L(a)||"string"===
typeof a)?null:Ya(a.replace(Aa,""),b,c)?!0:null},D=function(a,b,c){var d=[],e=0,f=a.length;if(c!==k)for(;e<f;e++)a[e]&&a[e][b]&&d.push(a[e][b][c]);else for(;e<f;e++)a[e]&&d.push(a[e][b]);return d},ia=function(a,b,c,d){var e=[],f=0,g=b.length;if(d!==k)for(;f<g;f++)a[b[f]][c]&&e.push(a[b[f]][c][d]);else for(;f<g;f++)e.push(a[b[f]][c]);return e},W=function(a,b){var c=[],d;b===k?(b=0,d=a):(d=b,b=a);for(var e=b;e<d;e++)c.push(e);return c},Qb=function(a){for(var b=[],c=0,d=a.length;c<d;c++)a[c]&&b.push(a[c]);
return b},qa=function(a){var b;a:{if(!(2>a.length)){b=a.slice().sort();for(var c=b[0],d=1,e=b.length;d<e;d++){if(b[d]===c){b=!1;break a}c=b[d]}}b=!0}if(b)return a.slice();b=[];var e=a.length,f,g=0,d=0;a:for(;d<e;d++){c=a[d];for(f=0;f<g;f++)if(b[f]===c)continue a;b.push(c);g++}return b};m.util={throttle:function(a,b){var c=b!==k?b:200,d,e;return function(){var b=this,g=+new Date,j=arguments;d&&g<d+c?(clearTimeout(e),e=setTimeout(function(){d=k;a.apply(b,j)},c)):(d=g,a.apply(b,j))}},escapeRegex:function(a){return a.replace($b,
"\\$1")}};var A=function(a,b,c){a[b]!==k&&(a[c]=a[b])},ba=/\[.*?\]$/,U=/\(\)$/,Pa=m.util.escapeRegex,va=h("<div>")[0],Wb=va.textContent!==k,Yb=/<.*?>/g,Na=m.util.throttle,Rb=[],w=Array.prototype,ac=function(a){var b,c,d=m.settings,e=h.map(d,function(a){return a.nTable});if(a){if(a.nTable&&a.oApi)return[a];if(a.nodeName&&"table"===a.nodeName.toLowerCase())return b=h.inArray(a,e),-1!==b?[d[b]]:null;if(a&&"function"===typeof a.settings)return a.settings().toArray();"string"===typeof a?c=h(a):a instanceof
h&&(c=a)}else return[];if(c)return c.map(function(){b=h.inArray(this,e);return-1!==b?d[b]:null}).toArray()};s=function(a,b){if(!(this instanceof s))return new s(a,b);var c=[],d=function(a){(a=ac(a))&&(c=c.concat(a))};if(h.isArray(a))for(var e=0,f=a.length;e<f;e++)d(a[e]);else d(a);this.context=qa(c);b&&h.merge(this,b);this.selector={rows:null,cols:null,opts:null};s.extend(this,this,Rb)};m.Api=s;h.extend(s.prototype,{any:function(){return 0!==this.count()},concat:w.concat,context:[],count:function(){return this.flatten().length},
each:function(a){for(var b=0,c=this.length;b<c;b++)a.call(this,this[b],b,this);return this},eq:function(a){var b=this.context;return b.length>a?new s(b[a],this[a]):null},filter:function(a){var b=[];if(w.filter)b=w.filter.call(this,a,this);else for(var c=0,d=this.length;c<d;c++)a.call(this,this[c],c,this)&&b.push(this[c]);return new s(this.context,b)},flatten:function(){var a=[];return new s(this.context,a.concat.apply(a,this.toArray()))},join:w.join,indexOf:w.indexOf||function(a,b){for(var c=b||0,
d=this.length;c<d;c++)if(this[c]===a)return c;return-1},iterator:function(a,b,c,d){var e=[],f,g,j,h,n,l=this.context,m,o,u=this.selector;"string"===typeof a&&(d=c,c=b,b=a,a=!1);g=0;for(j=l.length;g<j;g++){var r=new s(l[g]);if("table"===b)f=c.call(r,l[g],g),f!==k&&e.push(f);else if("columns"===b||"rows"===b)f=c.call(r,l[g],this[g],g),f!==k&&e.push(f);else if("column"===b||"column-rows"===b||"row"===b||"cell"===b){o=this[g];"column-rows"===b&&(m=Ba(l[g],u.opts));h=0;for(n=o.length;h<n;h++)f=o[h],f=
"cell"===b?c.call(r,l[g],f.row,f.column,g,h):c.call(r,l[g],f,g,h,m),f!==k&&e.push(f)}}return e.length||d?(a=new s(l,a?e.concat.apply([],e):e),b=a.selector,b.rows=u.rows,b.cols=u.cols,b.opts=u.opts,a):this},lastIndexOf:w.lastIndexOf||function(a,b){return this.indexOf.apply(this.toArray.reverse(),arguments)},length:0,map:function(a){var b=[];if(w.map)b=w.map.call(this,a,this);else for(var c=0,d=this.length;c<d;c++)b.push(a.call(this,this[c],c));return new s(this.context,b)},pluck:function(a){return this.map(function(b){return b[a]})},
pop:w.pop,push:w.push,reduce:w.reduce||function(a,b){return gb(this,a,b,0,this.length,1)},reduceRight:w.reduceRight||function(a,b){return gb(this,a,b,this.length-1,-1,-1)},reverse:w.reverse,selector:null,shift:w.shift,slice:function(){return new s(this.context,this)},sort:w.sort,splice:w.splice,toArray:function(){return w.slice.call(this)},to$:function(){return h(this)},toJQuery:function(){return h(this)},unique:function(){return new s(this.context,qa(this))},unshift:w.unshift});s.extend=function(a,
b,c){if(c.length&&b&&(b instanceof s||b.__dt_wrapper)){var d,e,f,g=function(a,b,c){return function(){var d=b.apply(a,arguments);s.extend(d,d,c.methodExt);return d}};d=0;for(e=c.length;d<e;d++)f=c[d],b[f.name]="function"===typeof f.val?g(a,f.val,f):h.isPlainObject(f.val)?{}:f.val,b[f.name].__dt_wrapper=!0,s.extend(a,b[f.name],f.propExt)}};s.register=o=function(a,b){if(h.isArray(a))for(var c=0,d=a.length;c<d;c++)s.register(a[c],b);else for(var e=a.split("."),f=Rb,g,j,c=0,d=e.length;c<d;c++){g=(j=-1!==
e[c].indexOf("()"))?e[c].replace("()",""):e[c];var i;a:{i=0;for(var n=f.length;i<n;i++)if(f[i].name===g){i=f[i];break a}i=null}i||(i={name:g,val:{},methodExt:[],propExt:[]},f.push(i));c===d-1?i.val=b:f=j?i.methodExt:i.propExt}};s.registerPlural=u=function(a,b,c){s.register(a,c);s.register(b,function(){var a=c.apply(this,arguments);return a===this?this:a instanceof s?a.length?h.isArray(a[0])?new s(a.context,a[0]):a[0]:k:a})};o("tables()",function(a){var b;if(a){b=s;var c=this.context;if("number"===
typeof a)a=[c[a]];else var d=h.map(c,function(a){return a.nTable}),a=h(d).filter(a).map(function(){var a=h.inArray(this,d);return c[a]}).toArray();b=new b(a)}else b=this;return b});o("table()",function(a){var a=this.tables(a),b=a.context;return b.length?new s(b[0]):a});u("tables().nodes()","table().node()",function(){return this.iterator("table",function(a){return a.nTable},1)});u("tables().body()","table().body()",function(){return this.iterator("table",function(a){return a.nTBody},1)});u("tables().header()",
"table().header()",function(){return this.iterator("table",function(a){return a.nTHead},1)});u("tables().footer()","table().footer()",function(){return this.iterator("table",function(a){return a.nTFoot},1)});u("tables().containers()","table().container()",function(){return this.iterator("table",function(a){return a.nTableWrapper},1)});o("draw()",function(a){return this.iterator("table",function(b){"page"===a?N(b):("string"===typeof a&&(a="full-hold"===a?!1:!0),S(b,!1===a))})});o("page()",function(a){return a===
k?this.page.info().page:this.iterator("table",function(b){Sa(b,a)})});o("page.info()",function(){if(0===this.context.length)return k;var a=this.context[0],b=a._iDisplayStart,c=a.oFeatures.bPaginate?a._iDisplayLength:-1,d=a.fnRecordsDisplay(),e=-1===c;return{page:e?0:Math.floor(b/c),pages:e?1:Math.ceil(d/c),start:b,end:a.fnDisplayEnd(),length:c,recordsTotal:a.fnRecordsTotal(),recordsDisplay:d,serverSide:"ssp"===y(a)}});o("page.len()",function(a){return a===k?0!==this.context.length?this.context[0]._iDisplayLength:
k:this.iterator("table",function(b){Qa(b,a)})});var Sb=function(a,b,c){if(c){var d=new s(a);d.one("draw",function(){c(d.ajax.json())})}if("ssp"==y(a))S(a,b);else{C(a,!0);var e=a.jqXHR;e&&4!==e.readyState&&e.abort();sa(a,[],function(c){na(a);for(var c=ta(a,c),d=0,e=c.length;d<e;d++)M(a,c[d]);S(a,b);C(a,!1)})}};o("ajax.json()",function(){var a=this.context;if(0<a.length)return a[0].json});o("ajax.params()",function(){var a=this.context;if(0<a.length)return a[0].oAjaxData});o("ajax.reload()",function(a,
b){return this.iterator("table",function(c){Sb(c,!1===b,a)})});o("ajax.url()",function(a){var b=this.context;if(a===k){if(0===b.length)return k;b=b[0];return b.ajax?h.isPlainObject(b.ajax)?b.ajax.url:b.ajax:b.sAjaxSource}return this.iterator("table",function(b){h.isPlainObject(b.ajax)?b.ajax.url=a:b.ajax=a})});o("ajax.url().load()",function(a,b){return this.iterator("table",function(c){Sb(c,!1===b,a)})});var Za=function(a,b,c,d,e){var f=[],g,j,i,n,l,m;i=typeof b;if(!b||"string"===i||"function"===
i||b.length===k)b=[b];i=0;for(n=b.length;i<n;i++){j=b[i]&&b[i].split&&!b[i].match(/[\[\(:]/)?b[i].split(","):[b[i]];l=0;for(m=j.length;l<m;l++)(g=c("string"===typeof j[l]?h.trim(j[l]):j[l]))&&g.length&&(f=f.concat(g))}a=x.selector[a];if(a.length){i=0;for(n=a.length;i<n;i++)f=a[i](d,e,f)}return qa(f)},$a=function(a){a||(a={});a.filter&&a.search===k&&(a.search=a.filter);return h.extend({search:"none",order:"current",page:"all"},a)},ab=function(a){for(var b=0,c=a.length;b<c;b++)if(0<a[b].length)return a[0]=
a[b],a[0].length=1,a.length=1,a.context=[a.context[b]],a;a.length=0;return a},Ba=function(a,b){var c,d,e,f=[],g=a.aiDisplay;c=a.aiDisplayMaster;var j=b.search;d=b.order;e=b.page;if("ssp"==y(a))return"removed"===j?[]:W(0,c.length);if("current"==e){c=a._iDisplayStart;for(d=a.fnDisplayEnd();c<d;c++)f.push(g[c])}else if("current"==d||"applied"==d)f="none"==j?c.slice():"applied"==j?g.slice():h.map(c,function(a){return-1===h.inArray(a,g)?a:null});else if("index"==d||"original"==d){c=0;for(d=a.aoData.length;c<
d;c++)"none"==j?f.push(c):(e=h.inArray(c,g),(-1===e&&"removed"==j||0<=e&&"applied"==j)&&f.push(c))}return f};o("rows()",function(a,b){a===k?a="":h.isPlainObject(a)&&(b=a,a="");var b=$a(b),c=this.iterator("table",function(c){var e=b,f;return Za("row",a,function(a){var b=Nb(a);if(b!==null&&!e)return[b];f||(f=Ba(c,e));if(b!==null&&h.inArray(b,f)!==-1)return[b];if(a===null||a===k||a==="")return f;if(typeof a==="function")return h.map(f,function(b){var e=c.aoData[b];return a(b,e._aData,e.nTr)?b:null});
b=Qb(ia(c.aoData,f,"nTr"));if(a.nodeName){if(a._DT_RowIndex!==k)return[a._DT_RowIndex];if(a._DT_CellIndex)return[a._DT_CellIndex.row];b=h(a).closest("*[data-dt-row]");return b.length?[b.data("dt-row")]:[]}if(typeof a==="string"&&a.charAt(0)==="#"){var i=c.aIds[a.replace(/^#/,"")];if(i!==k)return[i.idx]}return h(b).filter(a).map(function(){return this._DT_RowIndex}).toArray()},c,e)},1);c.selector.rows=a;c.selector.opts=b;return c});o("rows().nodes()",function(){return this.iterator("row",function(a,
b){return a.aoData[b].nTr||k},1)});o("rows().data()",function(){return this.iterator(!0,"rows",function(a,b){return ia(a.aoData,b,"_aData")},1)});u("rows().cache()","row().cache()",function(a){return this.iterator("row",function(b,c){var d=b.aoData[c];return"search"===a?d._aFilterData:d._aSortData},1)});u("rows().invalidate()","row().invalidate()",function(a){return this.iterator("row",function(b,c){ca(b,c,a)})});u("rows().indexes()","row().index()",function(){return this.iterator("row",function(a,
b){return b},1)});u("rows().ids()","row().id()",function(a){for(var b=[],c=this.context,d=0,e=c.length;d<e;d++)for(var f=0,g=this[d].length;f<g;f++){var h=c[d].rowIdFn(c[d].aoData[this[d][f]]._aData);b.push((!0===a?"#":"")+h)}return new s(c,b)});u("rows().remove()","row().remove()",function(){var a=this;this.iterator("row",function(b,c,d){var e=b.aoData,f=e[c],g,h,i,n,l;e.splice(c,1);g=0;for(h=e.length;g<h;g++)if(i=e[g],l=i.anCells,null!==i.nTr&&(i.nTr._DT_RowIndex=g),null!==l){i=0;for(n=l.length;i<
n;i++)l[i]._DT_CellIndex.row=g}oa(b.aiDisplayMaster,c);oa(b.aiDisplay,c);oa(a[d],c,!1);0<b._iRecordsDisplay&&b._iRecordsDisplay--;Ra(b);c=b.rowIdFn(f._aData);c!==k&&delete b.aIds[c]});this.iterator("table",function(a){for(var c=0,d=a.aoData.length;c<d;c++)a.aoData[c].idx=c});return this});o("rows.add()",function(a){var b=this.iterator("table",function(b){var c,f,g,h=[];f=0;for(g=a.length;f<g;f++)c=a[f],c.nodeName&&"TR"===c.nodeName.toUpperCase()?h.push(ma(b,c)[0]):h.push(M(b,c));return h},1),c=this.rows(-1);
c.pop();h.merge(c,b);return c});o("row()",function(a,b){return ab(this.rows(a,b))});o("row().data()",function(a){var b=this.context;if(a===k)return b.length&&this.length?b[0].aoData[this[0]]._aData:k;b[0].aoData[this[0]]._aData=a;ca(b[0],this[0],"data");return this});o("row().node()",function(){var a=this.context;return a.length&&this.length?a[0].aoData[this[0]].nTr||null:null});o("row.add()",function(a){a instanceof h&&a.length&&(a=a[0]);var b=this.iterator("table",function(b){return a.nodeName&&
"TR"===a.nodeName.toUpperCase()?ma(b,a)[0]:M(b,a)});return this.row(b[0])});var bb=function(a,b){var c=a.context;if(c.length&&(c=c[0].aoData[b!==k?b:a[0]])&&c._details)c._details.remove(),c._detailsShow=k,c._details=k},Tb=function(a,b){var c=a.context;if(c.length&&a.length){var d=c[0].aoData[a[0]];if(d._details){(d._detailsShow=b)?d._details.insertAfter(d.nTr):d._details.detach();var e=c[0],f=new s(e),g=e.aoData;f.off("draw.dt.DT_details column-visibility.dt.DT_details destroy.dt.DT_details");0<D(g,
"_details").length&&(f.on("draw.dt.DT_details",function(a,b){e===b&&f.rows({page:"current"}).eq(0).each(function(a){a=g[a];a._detailsShow&&a._details.insertAfter(a.nTr)})}),f.on("column-visibility.dt.DT_details",function(a,b){if(e===b)for(var c,d=aa(b),f=0,h=g.length;f<h;f++)c=g[f],c._details&&c._details.children("td[colspan]").attr("colspan",d)}),f.on("destroy.dt.DT_details",function(a,b){if(e===b)for(var c=0,d=g.length;c<d;c++)g[c]._details&&bb(f,c)}))}}};o("row().child()",function(a,b){var c=this.context;
if(a===k)return c.length&&this.length?c[0].aoData[this[0]]._details:k;if(!0===a)this.child.show();else if(!1===a)bb(this);else if(c.length&&this.length){var d=c[0],c=c[0].aoData[this[0]],e=[],f=function(a,b){if(h.isArray(a)||a instanceof h)for(var c=0,k=a.length;c<k;c++)f(a[c],b);else a.nodeName&&"tr"===a.nodeName.toLowerCase()?e.push(a):(c=h("<tr><td/></tr>").addClass(b),h("td",c).addClass(b).html(a)[0].colSpan=aa(d),e.push(c[0]))};f(a,b);c._details&&c._details.detach();c._details=h(e);c._detailsShow&&
c._details.insertAfter(c.nTr)}return this});o(["row().child.show()","row().child().show()"],function(){Tb(this,!0);return this});o(["row().child.hide()","row().child().hide()"],function(){Tb(this,!1);return this});o(["row().child.remove()","row().child().remove()"],function(){bb(this);return this});o("row().child.isShown()",function(){var a=this.context;return a.length&&this.length?a[0].aoData[this[0]]._detailsShow||!1:!1});var bc=/^([^:]+):(name|visIdx|visible)$/,Ub=function(a,b,c,d,e){for(var c=
[],d=0,f=e.length;d<f;d++)c.push(B(a,e[d],b));return c};o("columns()",function(a,b){a===k?a="":h.isPlainObject(a)&&(b=a,a="");var b=$a(b),c=this.iterator("table",function(c){var e=a,f=b,g=c.aoColumns,j=D(g,"sName"),i=D(g,"nTh");return Za("column",e,function(a){var b=Nb(a);if(a==="")return W(g.length);if(b!==null)return[b>=0?b:g.length+b];if(typeof a==="function"){var e=Ba(c,f);return h.map(g,function(b,f){return a(f,Ub(c,f,0,0,e),i[f])?f:null})}var k=typeof a==="string"?a.match(bc):"";if(k)switch(k[2]){case "visIdx":case "visible":b=
parseInt(k[1],10);if(b<0){var m=h.map(g,function(a,b){return a.bVisible?b:null});return[m[m.length+b]]}return[Z(c,b)];case "name":return h.map(j,function(a,b){return a===k[1]?b:null});default:return[]}if(a.nodeName&&a._DT_CellIndex)return[a._DT_CellIndex.column];b=h(i).filter(a).map(function(){return h.inArray(this,i)}).toArray();if(b.length||!a.nodeName)return b;b=h(a).closest("*[data-dt-column]");return b.length?[b.data("dt-column")]:[]},c,f)},1);c.selector.cols=a;c.selector.opts=b;return c});u("columns().header()",
"column().header()",function(){return this.iterator("column",function(a,b){return a.aoColumns[b].nTh},1)});u("columns().footer()","column().footer()",function(){return this.iterator("column",function(a,b){return a.aoColumns[b].nTf},1)});u("columns().data()","column().data()",function(){return this.iterator("column-rows",Ub,1)});u("columns().dataSrc()","column().dataSrc()",function(){return this.iterator("column",function(a,b){return a.aoColumns[b].mData},1)});u("columns().cache()","column().cache()",
function(a){return this.iterator("column-rows",function(b,c,d,e,f){return ia(b.aoData,f,"search"===a?"_aFilterData":"_aSortData",c)},1)});u("columns().nodes()","column().nodes()",function(){return this.iterator("column-rows",function(a,b,c,d,e){return ia(a.aoData,e,"anCells",b)},1)});u("columns().visible()","column().visible()",function(a,b){var c=this.iterator("column",function(b,c){if(a===k)return b.aoColumns[c].bVisible;var f=b.aoColumns,g=f[c],j=b.aoData,i,n,l;if(a!==k&&g.bVisible!==a){if(a){var m=
h.inArray(!0,D(f,"bVisible"),c+1);i=0;for(n=j.length;i<n;i++)l=j[i].nTr,f=j[i].anCells,l&&l.insertBefore(f[c],f[m]||null)}else h(D(b.aoData,"anCells",c)).detach();g.bVisible=a;ea(b,b.aoHeader);ea(b,b.aoFooter);xa(b)}});a!==k&&(this.iterator("column",function(c,e){r(c,null,"column-visibility",[c,e,a,b])}),(b===k||b)&&this.columns.adjust());return c});u("columns().indexes()","column().index()",function(a){return this.iterator("column",function(b,c){return"visible"===a?$(b,c):c},1)});o("columns.adjust()",
function(){return this.iterator("table",function(a){Y(a)},1)});o("column.index()",function(a,b){if(0!==this.context.length){var c=this.context[0];if("fromVisible"===a||"toData"===a)return Z(c,b);if("fromData"===a||"toVisible"===a)return $(c,b)}});o("column()",function(a,b){return ab(this.columns(a,b))});o("cells()",function(a,b,c){h.isPlainObject(a)&&(a.row===k?(c=a,a=null):(c=b,b=null));h.isPlainObject(b)&&(c=b,b=null);if(null===b||b===k)return this.iterator("table",function(b){var d=a,e=$a(c),f=
b.aoData,g=Ba(b,e),j=Qb(ia(f,g,"anCells")),i=h([].concat.apply([],j)),l,n=b.aoColumns.length,m,o,u,s,r,v;return Za("cell",d,function(a){var c=typeof a==="function";if(a===null||a===k||c){m=[];o=0;for(u=g.length;o<u;o++){l=g[o];for(s=0;s<n;s++){r={row:l,column:s};if(c){v=f[l];a(r,B(b,l,s),v.anCells?v.anCells[s]:null)&&m.push(r)}else m.push(r)}}return m}if(h.isPlainObject(a))return[a];c=i.filter(a).map(function(a,b){return{row:b._DT_CellIndex.row,column:b._DT_CellIndex.column}}).toArray();if(c.length||
!a.nodeName)return c;v=h(a).closest("*[data-dt-row]");return v.length?[{row:v.data("dt-row"),column:v.data("dt-column")}]:[]},b,e)});var d=this.columns(b,c),e=this.rows(a,c),f,g,j,i,n,l=this.iterator("table",function(a,b){f=[];g=0;for(j=e[b].length;g<j;g++){i=0;for(n=d[b].length;i<n;i++)f.push({row:e[b][g],column:d[b][i]})}return f},1);h.extend(l.selector,{cols:b,rows:a,opts:c});return l});u("cells().nodes()","cell().node()",function(){return this.iterator("cell",function(a,b,c){return(a=a.aoData[b])&&
a.anCells?a.anCells[c]:k},1)});o("cells().data()",function(){return this.iterator("cell",function(a,b,c){return B(a,b,c)},1)});u("cells().cache()","cell().cache()",function(a){a="search"===a?"_aFilterData":"_aSortData";return this.iterator("cell",function(b,c,d){return b.aoData[c][a][d]},1)});u("cells().render()","cell().render()",function(a){return this.iterator("cell",function(b,c,d){return B(b,c,d,a)},1)});u("cells().indexes()","cell().index()",function(){return this.iterator("cell",function(a,
b,c){return{row:b,column:c,columnVisible:$(a,c)}},1)});u("cells().invalidate()","cell().invalidate()",function(a){return this.iterator("cell",function(b,c,d){ca(b,c,a,d)})});o("cell()",function(a,b,c){return ab(this.cells(a,b,c))});o("cell().data()",function(a){var b=this.context,c=this[0];if(a===k)return b.length&&c.length?B(b[0],c[0].row,c[0].column):k;ib(b[0],c[0].row,c[0].column,a);ca(b[0],c[0].row,"data",c[0].column);return this});o("order()",function(a,b){var c=this.context;if(a===k)return 0!==
c.length?c[0].aaSorting:k;"number"===typeof a?a=[[a,b]]:a.length&&!h.isArray(a[0])&&(a=Array.prototype.slice.call(arguments));return this.iterator("table",function(b){b.aaSorting=a.slice()})});o("order.listener()",function(a,b,c){return this.iterator("table",function(d){La(d,a,b,c)})});o("order.fixed()",function(a){if(!a){var b=this.context,b=b.length?b[0].aaSortingFixed:k;return h.isArray(b)?{pre:b}:b}return this.iterator("table",function(b){b.aaSortingFixed=h.extend(!0,{},a)})});o(["columns().order()",
"column().order()"],function(a){var b=this;return this.iterator("table",function(c,d){var e=[];h.each(b[d],function(b,c){e.push([c,a])});c.aaSorting=e})});o("search()",function(a,b,c,d){var e=this.context;return a===k?0!==e.length?e[0].oPreviousSearch.sSearch:k:this.iterator("table",function(e){e.oFeatures.bFilter&&fa(e,h.extend({},e.oPreviousSearch,{sSearch:a+"",bRegex:null===b?!1:b,bSmart:null===c?!0:c,bCaseInsensitive:null===d?!0:d}),1)})});u("columns().search()","column().search()",function(a,
b,c,d){return this.iterator("column",function(e,f){var g=e.aoPreSearchCols;if(a===k)return g[f].sSearch;e.oFeatures.bFilter&&(h.extend(g[f],{sSearch:a+"",bRegex:null===b?!1:b,bSmart:null===c?!0:c,bCaseInsensitive:null===d?!0:d}),fa(e,e.oPreviousSearch,1))})});o("state()",function(){return this.context.length?this.context[0].oSavedState:null});o("state.clear()",function(){return this.iterator("table",function(a){a.fnStateSaveCallback.call(a.oInstance,a,{})})});o("state.loaded()",function(){return this.context.length?
this.context[0].oLoadedState:null});o("state.save()",function(){return this.iterator("table",function(a){xa(a)})});m.versionCheck=m.fnVersionCheck=function(a){for(var b=m.version.split("."),a=a.split("."),c,d,e=0,f=a.length;e<f;e++)if(c=parseInt(b[e],10)||0,d=parseInt(a[e],10)||0,c!==d)return c>d;return!0};m.isDataTable=m.fnIsDataTable=function(a){var b=h(a).get(0),c=!1;if(a instanceof m.Api)return!0;h.each(m.settings,function(a,e){var f=e.nScrollHead?h("table",e.nScrollHead)[0]:null,g=e.nScrollFoot?
h("table",e.nScrollFoot)[0]:null;if(e.nTable===b||f===b||g===b)c=!0});return c};m.tables=m.fnTables=function(a){var b=!1;h.isPlainObject(a)&&(b=a.api,a=a.visible);var c=h.map(m.settings,function(b){if(!a||a&&h(b.nTable).is(":visible"))return b.nTable});return b?new s(c):c};m.camelToHungarian=I;o("$()",function(a,b){var c=this.rows(b).nodes(),c=h(c);return h([].concat(c.filter(a).toArray(),c.find(a).toArray()))});h.each(["on","one","off"],function(a,b){o(b+"()",function(){var a=Array.prototype.slice.call(arguments);
a[0]=h.map(a[0].split(/\s/),function(a){return!a.match(/\.dt\b/)?a+".dt":a}).join(" ");var d=h(this.tables().nodes());d[b].apply(d,a);return this})});o("clear()",function(){return this.iterator("table",function(a){na(a)})});o("settings()",function(){return new s(this.context,this.context)});o("init()",function(){var a=this.context;return a.length?a[0].oInit:null});o("data()",function(){return this.iterator("table",function(a){return D(a.aoData,"_aData")}).flatten()});o("destroy()",function(a){a=a||
!1;return this.iterator("table",function(b){var c=b.nTableWrapper.parentNode,d=b.oClasses,e=b.nTable,f=b.nTBody,g=b.nTHead,j=b.nTFoot,i=h(e),f=h(f),k=h(b.nTableWrapper),l=h.map(b.aoData,function(a){return a.nTr}),o;b.bDestroying=!0;r(b,"aoDestroyCallback","destroy",[b]);a||(new s(b)).columns().visible(!0);k.off(".DT").find(":not(tbody *)").off(".DT");h(E).off(".DT-"+b.sInstance);e!=g.parentNode&&(i.children("thead").detach(),i.append(g));j&&e!=j.parentNode&&(i.children("tfoot").detach(),i.append(j));
b.aaSorting=[];b.aaSortingFixed=[];wa(b);h(l).removeClass(b.asStripeClasses.join(" "));h("th, td",g).removeClass(d.sSortable+" "+d.sSortableAsc+" "+d.sSortableDesc+" "+d.sSortableNone);f.children().detach();f.append(l);g=a?"remove":"detach";i[g]();k[g]();!a&&c&&(c.insertBefore(e,b.nTableReinsertBefore),i.css("width",b.sDestroyWidth).removeClass(d.sTable),(o=b.asDestroyStripes.length)&&f.children().each(function(a){h(this).addClass(b.asDestroyStripes[a%o])}));c=h.inArray(b,m.settings);-1!==c&&m.settings.splice(c,
1)})});h.each(["column","row","cell"],function(a,b){o(b+"s().every()",function(a){var d=this.selector.opts,e=this;return this.iterator(b,function(f,g,h,i,n){a.call(e[b](g,"cell"===b?h:d,"cell"===b?d:k),g,h,i,n)})})});o("i18n()",function(a,b,c){var d=this.context[0],a=Q(a)(d.oLanguage);a===k&&(a=b);c!==k&&h.isPlainObject(a)&&(a=a[c]!==k?a[c]:a._);return a.replace("%d",c)});m.version="1.10.16";m.settings=[];m.models={};m.models.oSearch={bCaseInsensitive:!0,sSearch:"",bRegex:!1,bSmart:!0};m.models.oRow=
{nTr:null,anCells:null,_aData:[],_aSortData:null,_aFilterData:null,_sFilterRow:null,_sRowStripe:"",src:null,idx:-1};m.models.oColumn={idx:null,aDataSort:null,asSorting:null,bSearchable:null,bSortable:null,bVisible:null,_sManualType:null,_bAttrSrc:!1,fnCreatedCell:null,fnGetData:null,fnSetData:null,mData:null,mRender:null,nTh:null,nTf:null,sClass:null,sContentPadding:null,sDefaultContent:null,sName:null,sSortDataType:"std",sSortingClass:null,sSortingClassJUI:null,sTitle:null,sType:null,sWidth:null,
sWidthOrig:null};m.defaults={aaData:null,aaSorting:[[0,"asc"]],aaSortingFixed:[],ajax:null,aLengthMenu:[10,25,50,100],aoColumns:null,aoColumnDefs:null,aoSearchCols:[],asStripeClasses:null,bAutoWidth:!0,bDeferRender:!1,bDestroy:!1,bFilter:!0,bInfo:!0,bLengthChange:!0,bPaginate:!0,bProcessing:!1,bRetrieve:!1,bScrollCollapse:!1,bServerSide:!1,bSort:!0,bSortMulti:!0,bSortCellsTop:!1,bSortClasses:!0,bStateSave:!1,fnCreatedRow:null,fnDrawCallback:null,fnFooterCallback:null,fnFormatNumber:function(a){return a.toString().replace(/\B(?=(\d{3})+(?!\d))/g,
this.oLanguage.sThousands)},fnHeaderCallback:null,fnInfoCallback:null,fnInitComplete:null,fnPreDrawCallback:null,fnRowCallback:null,fnServerData:null,fnServerParams:null,fnStateLoadCallback:function(a){try{return JSON.parse((-1===a.iStateDuration?sessionStorage:localStorage).getItem("DataTables_"+a.sInstance+"_"+location.pathname))}catch(b){}},fnStateLoadParams:null,fnStateLoaded:null,fnStateSaveCallback:function(a,b){try{(-1===a.iStateDuration?sessionStorage:localStorage).setItem("DataTables_"+a.sInstance+
"_"+location.pathname,JSON.stringify(b))}catch(c){}},fnStateSaveParams:null,iStateDuration:7200,iDeferLoading:null,iDisplayLength:10,iDisplayStart:0,iTabIndex:0,oClasses:{},oLanguage:{oAria:{sSortAscending:": activate to sort column ascending",sSortDescending:": activate to sort column descending"},oPaginate:{sFirst:"First",sLast:"Last",sNext:"Next",sPrevious:"Previous"},sEmptyTable:"No data available in table",sInfo:"Showing _START_ to _END_ of _TOTAL_ entries",sInfoEmpty:"Showing 0 to 0 of 0 entries",
sInfoFiltered:"(filtered from _MAX_ total entries)",sInfoPostFix:"",sDecimal:"",sThousands:",",sLengthMenu:"Show _MENU_ entries",sLoadingRecords:"Loading...",sProcessing:"Processing...",sSearch:"Search:",sSearchPlaceholder:"",sUrl:"",sZeroRecords:"No matching records found"},oSearch:h.extend({},m.models.oSearch),sAjaxDataProp:"data",sAjaxSource:null,sDom:"lfrtip",searchDelay:null,sPaginationType:"simple_numbers",sScrollX:"",sScrollXInner:"",sScrollY:"",sServerMethod:"GET",renderer:null,rowId:"DT_RowId"};
X(m.defaults);m.defaults.column={aDataSort:null,iDataSort:-1,asSorting:["asc","desc"],bSearchable:!0,bSortable:!0,bVisible:!0,fnCreatedCell:null,mData:null,mRender:null,sCellType:"td",sClass:"",sContentPadding:"",sDefaultContent:null,sName:"",sSortDataType:"std",sTitle:null,sType:null,sWidth:null};X(m.defaults.column);m.models.oSettings={oFeatures:{bAutoWidth:null,bDeferRender:null,bFilter:null,bInfo:null,bLengthChange:null,bPaginate:null,bProcessing:null,bServerSide:null,bSort:null,bSortMulti:null,
bSortClasses:null,bStateSave:null},oScroll:{bCollapse:null,iBarWidth:0,sX:null,sXInner:null,sY:null},oLanguage:{fnInfoCallback:null},oBrowser:{bScrollOversize:!1,bScrollbarLeft:!1,bBounding:!1,barWidth:0},ajax:null,aanFeatures:[],aoData:[],aiDisplay:[],aiDisplayMaster:[],aIds:{},aoColumns:[],aoHeader:[],aoFooter:[],oPreviousSearch:{},aoPreSearchCols:[],aaSorting:null,aaSortingFixed:[],asStripeClasses:null,asDestroyStripes:[],sDestroyWidth:0,aoRowCallback:[],aoHeaderCallback:[],aoFooterCallback:[],
aoDrawCallback:[],aoRowCreatedCallback:[],aoPreDrawCallback:[],aoInitComplete:[],aoStateSaveParams:[],aoStateLoadParams:[],aoStateLoaded:[],sTableId:"",nTable:null,nTHead:null,nTFoot:null,nTBody:null,nTableWrapper:null,bDeferLoading:!1,bInitialised:!1,aoOpenRows:[],sDom:null,searchDelay:null,sPaginationType:"two_button",iStateDuration:0,aoStateSave:[],aoStateLoad:[],oSavedState:null,oLoadedState:null,sAjaxSource:null,sAjaxDataProp:null,bAjaxDataGet:!0,jqXHR:null,json:k,oAjaxData:k,fnServerData:null,
aoServerParams:[],sServerMethod:null,fnFormatNumber:null,aLengthMenu:null,iDraw:0,bDrawing:!1,iDrawError:-1,_iDisplayLength:10,_iDisplayStart:0,_iRecordsTotal:0,_iRecordsDisplay:0,oClasses:{},bFiltered:!1,bSorted:!1,bSortCellsTop:null,oInit:null,aoDestroyCallback:[],fnRecordsTotal:function(){return"ssp"==y(this)?1*this._iRecordsTotal:this.aiDisplayMaster.length},fnRecordsDisplay:function(){return"ssp"==y(this)?1*this._iRecordsDisplay:this.aiDisplay.length},fnDisplayEnd:function(){var a=this._iDisplayLength,
b=this._iDisplayStart,c=b+a,d=this.aiDisplay.length,e=this.oFeatures,f=e.bPaginate;return e.bServerSide?!1===f||-1===a?b+d:Math.min(b+a,this._iRecordsDisplay):!f||c>d||-1===a?d:c},oInstance:null,sInstance:null,iTabIndex:0,nScrollHead:null,nScrollFoot:null,aLastSort:[],oPlugins:{},rowIdFn:null,rowId:null};m.ext=x={buttons:{},classes:{},builder:"-source-",errMode:"alert",feature:[],search:[],selector:{cell:[],column:[],row:[]},internal:{},legacy:{ajax:null},pager:{},renderer:{pageButton:{},header:{}},
order:{},type:{detect:[],search:{},order:{}},_unique:0,fnVersionCheck:m.fnVersionCheck,iApiIndex:0,oJUIClasses:{},sVersion:m.version};h.extend(x,{afnFiltering:x.search,aTypes:x.type.detect,ofnSearch:x.type.search,oSort:x.type.order,afnSortData:x.order,aoFeatures:x.feature,oApi:x.internal,oStdClasses:x.classes,oPagination:x.pager});h.extend(m.ext.classes,{sTable:"dataTable",sNoFooter:"no-footer",sPageButton:"paginate_button",sPageButtonActive:"current",sPageButtonDisabled:"disabled",sStripeOdd:"odd",
sStripeEven:"even",sRowEmpty:"dataTables_empty",sWrapper:"dataTables_wrapper",sFilter:"dataTables_filter",sInfo:"dataTables_info",sPaging:"dataTables_paginate paging_",sLength:"dataTables_length",sProcessing:"dataTables_processing",sSortAsc:"sorting_asc",sSortDesc:"sorting_desc",sSortable:"sorting",sSortableAsc:"sorting_asc_disabled",sSortableDesc:"sorting_desc_disabled",sSortableNone:"sorting_disabled",sSortColumn:"sorting_",sFilterInput:"",sLengthSelect:"",sScrollWrapper:"dataTables_scroll",sScrollHead:"dataTables_scrollHead",
sScrollHeadInner:"dataTables_scrollHeadInner",sScrollBody:"dataTables_scrollBody",sScrollFoot:"dataTables_scrollFoot",sScrollFootInner:"dataTables_scrollFootInner",sHeaderTH:"",sFooterTH:"",sSortJUIAsc:"",sSortJUIDesc:"",sSortJUI:"",sSortJUIAscAllowed:"",sSortJUIDescAllowed:"",sSortJUIWrapper:"",sSortIcon:"",sJUIHeader:"",sJUIFooter:""});var Kb=m.ext.pager;h.extend(Kb,{simple:function(){return["previous","next"]},full:function(){return["first","previous","next","last"]},numbers:function(a,b){return[ha(a,
b)]},simple_numbers:function(a,b){return["previous",ha(a,b),"next"]},full_numbers:function(a,b){return["first","previous",ha(a,b),"next","last"]},first_last_numbers:function(a,b){return["first",ha(a,b),"last"]},_numbers:ha,numbers_length:7});h.extend(!0,m.ext.renderer,{pageButton:{_:function(a,b,c,d,e,f){var g=a.oClasses,j=a.oLanguage.oPaginate,i=a.oLanguage.oAria.paginate||{},n,l,m=0,o=function(b,d){var k,s,u,r,v=function(b){Sa(a,b.data.action,true)};k=0;for(s=d.length;k<s;k++){r=d[k];if(h.isArray(r)){u=
h("<"+(r.DT_el||"div")+"/>").appendTo(b);o(u,r)}else{n=null;l="";switch(r){case "ellipsis":b.append('<span class="ellipsis">&#x2026;</span>');break;case "first":n=j.sFirst;l=r+(e>0?"":" "+g.sPageButtonDisabled);break;case "previous":n=j.sPrevious;l=r+(e>0?"":" "+g.sPageButtonDisabled);break;case "next":n=j.sNext;l=r+(e<f-1?"":" "+g.sPageButtonDisabled);break;case "last":n=j.sLast;l=r+(e<f-1?"":" "+g.sPageButtonDisabled);break;default:n=r+1;l=e===r?g.sPageButtonActive:""}if(n!==null){u=h("<a>",{"class":g.sPageButton+
" "+l,"aria-controls":a.sTableId,"aria-label":i[r],"data-dt-idx":m,tabindex:a.iTabIndex,id:c===0&&typeof r==="string"?a.sTableId+"_"+r:null}).html(n).appendTo(b);Va(u,{action:r},v);m++}}}},s;try{s=h(b).find(G.activeElement).data("dt-idx")}catch(u){}o(h(b).empty(),d);s!==k&&h(b).find("[data-dt-idx="+s+"]").focus()}}});h.extend(m.ext.type.detect,[function(a,b){var c=b.oLanguage.sDecimal;return Ya(a,c)?"num"+c:null},function(a){if(a&&!(a instanceof Date)&&!Zb.test(a))return null;var b=Date.parse(a);
return null!==b&&!isNaN(b)||L(a)?"date":null},function(a,b){var c=b.oLanguage.sDecimal;return Ya(a,c,!0)?"num-fmt"+c:null},function(a,b){var c=b.oLanguage.sDecimal;return Pb(a,c)?"html-num"+c:null},function(a,b){var c=b.oLanguage.sDecimal;return Pb(a,c,!0)?"html-num-fmt"+c:null},function(a){return L(a)||"string"===typeof a&&-1!==a.indexOf("<")?"html":null}]);h.extend(m.ext.type.search,{html:function(a){return L(a)?a:"string"===typeof a?a.replace(Mb," ").replace(Aa,""):""},string:function(a){return L(a)?
a:"string"===typeof a?a.replace(Mb," "):a}});var za=function(a,b,c,d){if(0!==a&&(!a||"-"===a))return-Infinity;b&&(a=Ob(a,b));a.replace&&(c&&(a=a.replace(c,"")),d&&(a=a.replace(d,"")));return 1*a};h.extend(x.type.order,{"date-pre":function(a){return Date.parse(a)||-Infinity},"html-pre":function(a){return L(a)?"":a.replace?a.replace(/<.*?>/g,"").toLowerCase():a+""},"string-pre":function(a){return L(a)?"":"string"===typeof a?a.toLowerCase():!a.toString?"":a.toString()},"string-asc":function(a,b){return a<
b?-1:a>b?1:0},"string-desc":function(a,b){return a<b?1:a>b?-1:0}});cb("");h.extend(!0,m.ext.renderer,{header:{_:function(a,b,c,d){h(a.nTable).on("order.dt.DT",function(e,f,g,h){if(a===f){e=c.idx;b.removeClass(c.sSortingClass+" "+d.sSortAsc+" "+d.sSortDesc).addClass(h[e]=="asc"?d.sSortAsc:h[e]=="desc"?d.sSortDesc:c.sSortingClass)}})},jqueryui:function(a,b,c,d){h("<div/>").addClass(d.sSortJUIWrapper).append(b.contents()).append(h("<span/>").addClass(d.sSortIcon+" "+c.sSortingClassJUI)).appendTo(b);
h(a.nTable).on("order.dt.DT",function(e,f,g,h){if(a===f){e=c.idx;b.removeClass(d.sSortAsc+" "+d.sSortDesc).addClass(h[e]=="asc"?d.sSortAsc:h[e]=="desc"?d.sSortDesc:c.sSortingClass);b.find("span."+d.sSortIcon).removeClass(d.sSortJUIAsc+" "+d.sSortJUIDesc+" "+d.sSortJUI+" "+d.sSortJUIAscAllowed+" "+d.sSortJUIDescAllowed).addClass(h[e]=="asc"?d.sSortJUIAsc:h[e]=="desc"?d.sSortJUIDesc:c.sSortingClassJUI)}})}}});var Vb=function(a){return"string"===typeof a?a.replace(/</g,"&lt;").replace(/>/g,"&gt;").replace(/"/g,
"&quot;"):a};m.render={number:function(a,b,c,d,e){return{display:function(f){if("number"!==typeof f&&"string"!==typeof f)return f;var g=0>f?"-":"",h=parseFloat(f);if(isNaN(h))return Vb(f);h=h.toFixed(c);f=Math.abs(h);h=parseInt(f,10);f=c?b+(f-h).toFixed(c).substring(2):"";return g+(d||"")+h.toString().replace(/\B(?=(\d{3})+(?!\d))/g,a)+f+(e||"")}}},text:function(){return{display:Vb}}};h.extend(m.ext.internal,{_fnExternApiFunc:Lb,_fnBuildAjax:sa,_fnAjaxUpdate:kb,_fnAjaxParameters:tb,_fnAjaxUpdateDraw:ub,
_fnAjaxDataSrc:ta,_fnAddColumn:Da,_fnColumnOptions:ja,_fnAdjustColumnSizing:Y,_fnVisibleToColumnIndex:Z,_fnColumnIndexToVisible:$,_fnVisbleColumns:aa,_fnGetColumns:la,_fnColumnTypes:Fa,_fnApplyColumnDefs:hb,_fnHungarianMap:X,_fnCamelToHungarian:I,_fnLanguageCompat:Ca,_fnBrowserDetect:fb,_fnAddData:M,_fnAddTr:ma,_fnNodeToDataIndex:function(a,b){return b._DT_RowIndex!==k?b._DT_RowIndex:null},_fnNodeToColumnIndex:function(a,b,c){return h.inArray(c,a.aoData[b].anCells)},_fnGetCellData:B,_fnSetCellData:ib,
_fnSplitObjNotation:Ia,_fnGetObjectDataFn:Q,_fnSetObjectDataFn:R,_fnGetDataMaster:Ja,_fnClearTable:na,_fnDeleteIndex:oa,_fnInvalidate:ca,_fnGetRowElements:Ha,_fnCreateTr:Ga,_fnBuildHead:jb,_fnDrawHead:ea,_fnDraw:N,_fnReDraw:S,_fnAddOptionsHtml:mb,_fnDetectHeader:da,_fnGetUniqueThs:ra,_fnFeatureHtmlFilter:ob,_fnFilterComplete:fa,_fnFilterCustom:xb,_fnFilterColumn:wb,_fnFilter:vb,_fnFilterCreateSearch:Oa,_fnEscapeRegex:Pa,_fnFilterData:yb,_fnFeatureHtmlInfo:rb,_fnUpdateInfo:Bb,_fnInfoMacros:Cb,_fnInitialise:ga,
_fnInitComplete:ua,_fnLengthChange:Qa,_fnFeatureHtmlLength:nb,_fnFeatureHtmlPaginate:sb,_fnPageChange:Sa,_fnFeatureHtmlProcessing:pb,_fnProcessingDisplay:C,_fnFeatureHtmlTable:qb,_fnScrollDraw:ka,_fnApplyToChildren:H,_fnCalculateColumnWidths:Ea,_fnThrottle:Na,_fnConvertToWidth:Db,_fnGetWidestNode:Eb,_fnGetMaxLenString:Fb,_fnStringToCss:v,_fnSortFlatten:V,_fnSort:lb,_fnSortAria:Hb,_fnSortListener:Ua,_fnSortAttachListener:La,_fnSortingClasses:wa,_fnSortData:Gb,_fnSaveState:xa,_fnLoadState:Ib,_fnSettingsFromNode:ya,
_fnLog:J,_fnMap:F,_fnBindAction:Va,_fnCallbackReg:z,_fnCallbackFire:r,_fnLengthOverflow:Ra,_fnRenderer:Ma,_fnDataSource:y,_fnRowAttributes:Ka,_fnCalculateEnd:function(){}});h.fn.dataTable=m;m.$=h;h.fn.dataTableSettings=m.settings;h.fn.dataTableExt=m.ext;h.fn.DataTable=function(a){return h(this).dataTable(a).api()};h.each(m,function(a,b){h.fn.DataTable[a]=b});return h.fn.dataTable});
</script>
<style type="text/css">
.container-fluid.crosstalk-bscols {
margin-left: -30px;
margin-right: -30px;
white-space: normal;
}

body > .container-fluid.crosstalk-bscols {
margin-left: auto;
margin-right: auto;
}
.crosstalk-input-checkboxgroup .crosstalk-options-group .crosstalk-options-column {
display: inline-block;
padding-right: 12px;
vertical-align: top;
}
@media only screen and (max-width:480px) {
.crosstalk-input-checkboxgroup .crosstalk-options-group .crosstalk-options-column {
display: block;
padding-right: inherit;
}
}
</style>
<script>!function a(b,c,d){function e(g,h){if(!c[g]){if(!b[g]){var i="function"==typeof require&&require;if(!h&&i)return i(g,!0);if(f)return f(g,!0);var j=new Error("Cannot find module '"+g+"'");throw j.code="MODULE_NOT_FOUND",j}var k=c[g]={exports:{}};b[g][0].call(k.exports,function(a){var c=b[g][1][a];return e(c?c:a)},k,k.exports,a,b,c,d)}return c[g].exports}for(var f="function"==typeof require&&require,g=0;g<d.length;g++)e(d[g]);return e}({1:[function(a,b,c){"use strict";function d(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}Object.defineProperty(c,"__esModule",{value:!0});var e=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),f=function(){function a(){d(this,a),this._types={},this._seq=0}return e(a,[{key:"on",value:function(a,b){var c=this._types[a];c||(c=this._types[a]={});var d="sub"+this._seq++;return c[d]=b,d}},{key:"off",value:function(a,b){var c=this._types[a];if("function"==typeof b){for(var d in c)if(c.hasOwnProperty(d)&&c[d]===b)return delete c[d],d;return!1}if("string"==typeof b)return!(!c||!c[b])&&(delete c[b],b);throw new Error("Unexpected type for listener")}},{key:"trigger",value:function(a,b,c){var d=this._types[a];for(var e in d)d.hasOwnProperty(e)&&d[e].call(c,b)}}]),a}();c.default=f},{}],2:[function(a,b,c){"use strict";function d(a){if(a&&a.__esModule)return a;var b={};if(null!=a)for(var c in a)Object.prototype.hasOwnProperty.call(a,c)&&(b[c]=a[c]);return b.default=a,b}function e(a){return a&&a.__esModule?a:{default:a}}function f(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}function g(a){var b=a.var("filterset"),c=b.get();return c||(c=new m.default,b.set(c)),c}function h(){return r++}Object.defineProperty(c,"__esModule",{value:!0}),c.FilterHandle=void 0;var i=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),j=a("./events"),k=e(j),l=a("./filterset"),m=e(l),n=a("./group"),o=e(n),p=a("./util"),q=d(p),r=1;c.FilterHandle=function(){function a(b,c){f(this,a),this._eventRelay=new k.default,this._emitter=new q.SubscriptionTracker(this._eventRelay),this._group=null,this._filterSet=null,this._filterVar=null,this._varOnChangeSub=null,this._extraInfo=q.extend({sender:this},c),this._id="filter"+h(),this.setGroup(b)}return i(a,[{key:"setGroup",value:function(a){var b=this;if(this._group!==a&&(this._group||a)&&(this._filterVar&&(this._filterVar.off("change",this._varOnChangeSub),this.clear(),this._varOnChangeSub=null,this._filterVar=null,this._filterSet=null),this._group=a,a)){a=(0,o.default)(a),this._filterSet=g(a),this._filterVar=(0,o.default)(a).var("filter");var c=this._filterVar.on("change",function(a){b._eventRelay.trigger("change",a,b)});this._varOnChangeSub=c}}},{key:"_mergeExtraInfo",value:function(a){return q.extend({},this._extraInfo?this._extraInfo:null,a?a:null)}},{key:"close",value:function(){this._emitter.removeAllListeners(),this.clear(),this.setGroup(null)}},{key:"clear",value:function(a){this._filterSet&&(this._filterSet.clear(this._id),this._onChange(a))}},{key:"set",value:function(a,b){this._filterSet&&(this._filterSet.update(this._id,a),this._onChange(b))}},{key:"on",value:function(a,b){return this._emitter.on(a,b)}},{key:"off",value:function(a,b){return this._emitter.off(a,b)}},{key:"_onChange",value:function(a){this._filterSet&&this._filterVar.set(this._filterSet.value,this._mergeExtraInfo(a))}},{key:"filteredKeys",get:function(){return this._filterSet?this._filterSet.value:null}}]),a}()},{"./events":1,"./filterset":3,"./group":4,"./util":11}],3:[function(a,b,c){"use strict";function d(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}function e(a,b){return a===b?0:a<b?-1:a>b?1:void 0}Object.defineProperty(c,"__esModule",{value:!0});var f=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),g=a("./util"),h=function(){function a(){d(this,a),this.reset()}return f(a,[{key:"reset",value:function(){this._handles={},this._keys={},this._value=null,this._activeHandles=0}},{key:"update",value:function(a,b){null!==b&&(b=b.slice(0),b.sort(e));var c=(0,g.diffSortedLists)(this._handles[a],b),d=c.added,f=c.removed;this._handles[a]=b;for(var h=0;h<d.length;h++)this._keys[d[h]]=(this._keys[d[h]]||0)+1;for(var i=0;i<f.length;i++)this._keys[f[i]]--;this._updateValue(b)}},{key:"_updateValue",value:function(){var a=arguments.length>0&&void 0!==arguments[0]?arguments[0]:this._allKeys,b=Object.keys(this._handles).length;if(0===b)this._value=null;else{this._value=[];for(var c=0;c<a.length;c++){var d=this._keys[a[c]];d===b&&this._value.push(a[c])}}}},{key:"clear",value:function(a){if("undefined"!=typeof this._handles[a]){var b=this._handles[a];b||(b=[]);for(var c=0;c<b.length;c++)this._keys[b[c]]--;delete this._handles[a],this._updateValue()}}},{key:"value",get:function(){return this._value}},{key:"_allKeys",get:function(){var a=Object.keys(this._keys);return a.sort(e),a}}]),a}();c.default=h},{"./util":11}],4:[function(a,b,c){(function(b){"use strict";function d(a){return a&&a.__esModule?a:{default:a}}function e(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}function f(a){if(a&&"string"==typeof a)return k.hasOwnProperty(a)||(k[a]=new l(a)),k[a];if("object"===("undefined"==typeof a?"undefined":h(a))&&a._vars&&a.var)return a;if(Array.isArray(a)&&1==a.length&&"string"==typeof a[0])return f(a[0]);throw new Error("Invalid groupName argument")}Object.defineProperty(c,"__esModule",{value:!0});var g=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),h="function"==typeof Symbol&&"symbol"==typeof Symbol.iterator?function(a){return typeof a}:function(a){return a&&"function"==typeof Symbol&&a.constructor===Symbol&&a!==Symbol.prototype?"symbol":typeof a};c.default=f;var i=a("./var"),j=d(i);b.__crosstalk_groups=b.__crosstalk_groups||{};var k=b.__crosstalk_groups,l=function(){function a(b){e(this,a),this.name=b,this._vars={}}return g(a,[{key:"var",value:function(a){if(!a||"string"!=typeof a)throw new Error("Invalid var name");return this._vars.hasOwnProperty(a)||(this._vars[a]=new j.default(this,a)),this._vars[a]}},{key:"has",value:function(a){if(!a||"string"!=typeof a)throw new Error("Invalid var name");return this._vars.hasOwnProperty(a)}}]),a}()}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{"./var":12}],5:[function(a,b,c){(function(b){"use strict";function d(a){return a&&a.__esModule?a:{default:a}}function e(a){return k.var(a)}function f(a){return k.has(a)}Object.defineProperty(c,"__esModule",{value:!0});var g=a("./group"),h=d(g),i=a("./selection"),j=a("./filter");a("./input"),a("./input_selectize"),a("./input_checkboxgroup"),a("./input_slider");var k=(0,h.default)("default");b.Shiny&&b.Shiny.addCustomMessageHandler("update-client-value",function(a){"string"==typeof a.group?(0,h.default)(a.group).var(a.name).set(a.value):e(a.name).set(a.value)});var l={group:h.default,var:e,has:f,SelectionHandle:i.SelectionHandle,FilterHandle:j.FilterHandle};c.default=l,b.crosstalk=l}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{"./filter":2,"./group":4,"./input":6,"./input_checkboxgroup":7,"./input_selectize":8,"./input_slider":9,"./selection":10}],6:[function(a,b,c){(function(a){"use strict";function b(b){i[b.className]=b,a.document&&"complete"!==a.document.readyState?h(function(){d()}):a.document&&setTimeout(d,100)}function d(){Object.keys(i).forEach(function(a){var b=i[a];h("."+b.className).not(".crosstalk-input-bound").each(function(a,c){g(b,c)})})}function e(a){return a.replace(/([!"#$%&'()*+,.\/:;<=>?@\[\\\]^`{|}~])/g,"\\$1")}function f(a){var b=h(a);Object.keys(i).forEach(function(c){if(b.hasClass(c)&&!b.hasClass("crosstalk-input-bound")){var d=i[c];g(d,a)}})}function g(a,b){var c=h(b).find("script[type='application/json'][data-for='"+e(b.id)+"']"),d=JSON.parse(c[0].innerText),f=a.factory(b,d);h(b).data("crosstalk-instance",f),h(b).addClass("crosstalk-input-bound")}Object.defineProperty(c,"__esModule",{value:!0}),c.register=b;var h=a.jQuery,i={};a.Shiny&&!function(){var b=new a.Shiny.InputBinding,c=a.jQuery;c.extend(b,{find:function(a){return c(a).find(".crosstalk-input")},initialize:function(a){c(a).hasClass("crosstalk-input-bound")||f(a)},getId:function(a){return a.id},getValue:function(a){},setValue:function(a,b){},receiveMessage:function(a,b){},subscribe:function(a,b){c(a).data("crosstalk-instance").resume()},unsubscribe:function(a){c(a).data("crosstalk-instance").suspend()}}),a.Shiny.inputBindings.register(b,"crosstalk.inputBinding")}()}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{}],7:[function(a,b,c){(function(b){"use strict";function c(a){if(a&&a.__esModule)return a;var b={};if(null!=a)for(var c in a)Object.prototype.hasOwnProperty.call(a,c)&&(b[c]=a[c]);return b.default=a,b}var d=a("./input"),e=c(d),f=a("./filter"),g=b.jQuery;e.register({className:"crosstalk-input-checkboxgroup",factory:function(a,b){var c=new f.FilterHandle(b.group),d=void 0,e=g(a);return e.on("change","input[type='checkbox']",function(){var a=e.find("input[type='checkbox']:checked");0===a.length?(d=null,c.clear()):!function(){var e={};a.each(function(){b.map[this.value].forEach(function(a){e[a]=!0})});var f=Object.keys(e);f.sort(),d=f,c.set(f)}()}),{suspend:function(){c.clear()},resume:function(){d&&c.set(d)}}}})}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{"./filter":2,"./input":6}],8:[function(a,b,c){(function(b){"use strict";function c(a){if(a&&a.__esModule)return a;var b={};if(null!=a)for(var c in a)Object.prototype.hasOwnProperty.call(a,c)&&(b[c]=a[c]);return b.default=a,b}var d=a("./input"),e=c(d),f=a("./util"),g=c(f),h=a("./filter"),i=b.jQuery;e.register({className:"crosstalk-input-select",factory:function(a,b){var c=[{value:"",label:"(All)"}],d=g.dataframeToD3(b.items),e={options:c.concat(d),valueField:"value",labelField:"label",searchField:"label"},f=i(a).find("select")[0],j=i(f).selectize(e)[0].selectize,k=new h.FilterHandle(b.group),l=void 0;return j.on("change",function(){0===j.items.length?(l=null,k.clear()):!function(){var a={};j.items.forEach(function(c){b.map[c].forEach(function(b){a[b]=!0})});var c=Object.keys(a);c.sort(),l=c,k.set(c)}()}),{suspend:function(){k.clear()},resume:function(){l&&k.set(l)}}}})}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{"./filter":2,"./input":6,"./util":11}],9:[function(a,b,c){(function(b){"use strict";function c(a){if(a&&a.__esModule)return a;var b={};if(null!=a)for(var c in a)Object.prototype.hasOwnProperty.call(a,c)&&(b[c]=a[c]);return b.default=a,b}function d(a,b){for(var c=a.toString();c.length<b;)c="0"+c;return c}function e(a){return a instanceof Date?a.getUTCFullYear()+"-"+d(a.getUTCMonth()+1,2)+"-"+d(a.getUTCDate(),2):null}var f=function(){function a(a,b){var c=[],d=!0,e=!1,f=void 0;try{for(var g,h=a[Symbol.iterator]();!(d=(g=h.next()).done)&&(c.push(g.value),!b||c.length!==b);d=!0);}catch(a){e=!0,f=a}finally{try{!d&&h.return&&h.return()}finally{if(e)throw f}}return c}return function(b,c){if(Array.isArray(b))return b;if(Symbol.iterator in Object(b))return a(b,c);throw new TypeError("Invalid attempt to destructure non-iterable instance")}}(),g=a("./input"),h=c(g),i=a("./filter"),j=b.jQuery,k=b.strftime;h.register({className:"crosstalk-input-slider",factory:function(a,b){function c(){var a=h.data("ionRangeSlider").result,b=void 0,c=h.data("data-type");return b="date"===c?function(a){return e(new Date(+a))}:"datetime"===c?function(a){return+a/1e3}:function(a){return+a},"double"===h.data("ionRangeSlider").options.type?[b(a.from),b(a.to)]:b(a.from)}var d=new i.FilterHandle(b.group),g={},h=j(a).find("input"),l=h.data("data-type"),m=h.data("time-format"),n=void 0;if("date"===l)n=k.utc(),g.prettify=function(a){return n(m,new Date(a))};else if("datetime"===l){var o=h.data("timezone");n=o?k.timezone(o):k,g.prettify=function(a){return n(m,new Date(a))}}h.ionRangeSlider(g);var p=null;return h.on("change.crosstalkSliderInput",function(a){if(!h.data("updating")&&!h.data("animating")){for(var e=c(),g=f(e,2),i=g[0],j=g[1],k=[],l=0;l<b.values.length;l++){var m=b.values[l];m>=i&&m<=j&&k.push(b.keys[l])}k.sort(),d.set(k),p=k}}),{suspend:function(){d.clear()},resume:function(){p&&d.set(p)}}}})}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{"./filter":2,"./input":6}],10:[function(a,b,c){"use strict";function d(a){if(a&&a.__esModule)return a;var b={};if(null!=a)for(var c in a)Object.prototype.hasOwnProperty.call(a,c)&&(b[c]=a[c]);return b.default=a,b}function e(a){return a&&a.__esModule?a:{default:a}}function f(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}Object.defineProperty(c,"__esModule",{value:!0}),c.SelectionHandle=void 0;var g=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),h=a("./events"),i=e(h),j=a("./group"),k=e(j),l=a("./util"),m=d(l);c.SelectionHandle=function(){function a(){var b=arguments.length>0&&void 0!==arguments[0]?arguments[0]:null,c=arguments.length>1&&void 0!==arguments[1]?arguments[1]:null;f(this,a),this._eventRelay=new i.default,this._emitter=new m.SubscriptionTracker(this._eventRelay),this._group=null,this._var=null,this._varOnChangeSub=null,this._extraInfo=m.extend({sender:this},c),this.setGroup(b)}return g(a,[{key:"setGroup",value:function(a){var b=this;if(this._group!==a&&(this._group||a)&&(this._var&&(this._var.off("change",this._varOnChangeSub),this._var=null,this._varOnChangeSub=null),this._group=a,a)){this._var=(0,k.default)(a).var("selection");var c=this._var.on("change",function(a){b._eventRelay.trigger("change",a,b)});this._varOnChangeSub=c}}},{key:"_mergeExtraInfo",value:function(a){return m.extend({},this._extraInfo?this._extraInfo:null,a?a:null)}},{key:"set",value:function(a,b){this._var&&this._var.set(a,this._mergeExtraInfo(b))}},{key:"clear",value:function(a){this._var&&this.set(void 0,this._mergeExtraInfo(a))}},{key:"on",value:function(a,b){return this._emitter.on(a,b)}},{key:"off",value:function(a,b){return this._emitter.off(a,b)}},{key:"close",value:function(){this._emitter.removeAllListeners(),this.setGroup(null)}},{key:"value",get:function(){return this._var?this._var.get():null}}]),a}()},{"./events":1,"./group":4,"./util":11}],11:[function(a,b,c){"use strict";function d(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}function e(a){for(var b=arguments.length,c=Array(b>1?b-1:0),d=1;d<b;d++)c[d-1]=arguments[d];for(var e=0;e<c.length;e++){var f=c[e];if("undefined"!=typeof f&&null!==f)for(var g in f)f.hasOwnProperty(g)&&(a[g]=f[g])}return a}function f(a){for(var b=1;b<a.length;b++)if(a[b]<=a[b-1])throw new Error("List is not sorted or contains duplicate")}function g(a,b){var c=0,d=0;a||(a=[]),b||(b=[]);var e=[],g=[];for(f(a),f(b);c<a.length&&d<b.length;)a[c]===b[d]?(c++,d++):a[c]<b[d]?e.push(a[c++]):g.push(b[d++]);return c<a.length&&(e=e.concat(a.slice(c))),d<b.length&&(g=g.concat(b.slice(d))),{removed:e,added:g}}function h(a){var b=[],c=void 0;for(var d in a){if(a.hasOwnProperty(d)&&b.push(d),"object"!==j(a[d])||"undefined"==typeof a[d].length)throw new Error("All fields must be arrays");if("undefined"!=typeof c&&c!==a[d].length)throw new Error("All fields must be arrays of the same length");c=a[d].length}for(var e=[],f=void 0,g=0;g<c;g++){f={};for(var h=0;h<b.length;h++)f[b[h]]=a[b[h]][g];e.push(f)}return e}Object.defineProperty(c,"__esModule",{value:!0});var i=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),j="function"==typeof Symbol&&"symbol"==typeof Symbol.iterator?function(a){return typeof a}:function(a){return a&&"function"==typeof Symbol&&a.constructor===Symbol&&a!==Symbol.prototype?"symbol":typeof a};c.extend=e,c.checkSorted=f,c.diffSortedLists=g,c.dataframeToD3=h;c.SubscriptionTracker=function(){function a(b){d(this,a),this._emitter=b,this._subs={}}return i(a,[{key:"on",value:function(a,b){var c=this._emitter.on(a,b);return this._subs[c]=a,c}},{key:"off",value:function(a,b){var c=this._emitter.off(a,b);return c&&delete this._subs[c],c}},{key:"removeAllListeners",value:function(){var a=this,b=this._subs;this._subs={},Object.keys(b).forEach(function(c){a._emitter.off(b[c],c)})}}]),a}()},{}],12:[function(a,b,c){(function(b){"use strict";function d(a){return a&&a.__esModule?a:{default:a}}function e(a,b){if(!(a instanceof b))throw new TypeError("Cannot call a class as a function")}Object.defineProperty(c,"__esModule",{value:!0});var f="function"==typeof Symbol&&"symbol"==typeof Symbol.iterator?function(a){return typeof a}:function(a){return a&&"function"==typeof Symbol&&a.constructor===Symbol&&a!==Symbol.prototype?"symbol":typeof a},g=function(){function a(a,b){for(var c=0;c<b.length;c++){var d=b[c];d.enumerable=d.enumerable||!1,d.configurable=!0,"value"in d&&(d.writable=!0),Object.defineProperty(a,d.key,d)}}return function(b,c,d){return c&&a(b.prototype,c),d&&a(b,d),b}}(),h=a("./events"),i=d(h),j=function(){function a(b,c,d){e(this,a),this._group=b,this._name=c,this._value=d,this._events=new i.default}return g(a,[{key:"get",value:function(){return this._value}},{key:"set",value:function(a,c){if(this._value!==a){var d=this._value;this._value=a;var e={};if(c&&"object"===("undefined"==typeof c?"undefined":f(c)))for(var g in c)c.hasOwnProperty(g)&&(e[g]=c[g]);e.oldValue=d,e.value=a,this._events.trigger("change",e,this),b.Shiny&&b.Shiny.onInputChange&&b.Shiny.onInputChange(".clientValue-"+(null!==this._group.name?this._group.name+"-":"")+this._name,"undefined"==typeof a?null:a)}}},{key:"on",value:function(a,b){return this._events.on(a,b)}},{key:"off",value:function(a,b){return this._events.off(a,b)}}]),a}();c.default=j}).call(this,"undefined"!=typeof global?global:"undefined"!=typeof self?self:"undefined"!=typeof window?window:{})},{"./events":1}]},{},[5]);
//# sourceMappingURL=crosstalk.min.js.map</script>


<style type="text/css">code{white-space: pre;}</style>
<style type="text/css" data-origin="pandoc">
a.sourceLine { display: inline-block; line-height: 1.25; }
a.sourceLine { pointer-events: none; color: inherit; text-decoration: inherit; }
a.sourceLine:empty { height: 1.2em; }
.sourceCode { overflow: visible; }
code.sourceCode { white-space: pre; position: relative; }
div.sourceCode { margin: 1em 0; }
pre.sourceCode { margin: 0; }
@media screen {
div.sourceCode { overflow: auto; }
}
@media print {
code.sourceCode { white-space: pre-wrap; }
a.sourceLine { text-indent: -1em; padding-left: 1em; }
}
pre.numberSource a.sourceLine
  { position: relative; left: -4em; }
pre.numberSource a.sourceLine::before
  { content: attr(data-line-number);
    position: relative; left: -1em; text-align: right; vertical-align: baseline;
    border: none; pointer-events: all; display: inline-block;
    -webkit-touch-callout: none; -webkit-user-select: none;
    -khtml-user-select: none; -moz-user-select: none;
    -ms-user-select: none; user-select: none;
    padding: 0 4px; width: 4em;
    color: #aaaaaa;
  }
pre.numberSource { margin-left: 3em; border-left: 1px solid #aaaaaa;  padding-left: 4px; }
div.sourceCode
  {  }
@media screen {
a.sourceLine::before { text-decoration: underline; }
}
code span.al { color: #ff0000; font-weight: bold; } /* Alert */
code span.an { color: #60a0b0; font-weight: bold; font-style: italic; } /* Annotation */
code span.at { color: #7d9029; } /* Attribute */
code span.bn { color: #40a070; } /* BaseN */
code span.bu { } /* BuiltIn */
code span.cf { color: #007020; font-weight: bold; } /* ControlFlow */
code span.ch { color: #4070a0; } /* Char */
code span.cn { color: #880000; } /* Constant */
code span.co { color: #60a0b0; font-style: italic; } /* Comment */
code span.cv { color: #60a0b0; font-weight: bold; font-style: italic; } /* CommentVar */
code span.do { color: #ba2121; font-style: italic; } /* Documentation */
code span.dt { color: #902000; } /* DataType */
code span.dv { color: #40a070; } /* DecVal */
code span.er { color: #ff0000; font-weight: bold; } /* Error */
code span.ex { } /* Extension */
code span.fl { color: #40a070; } /* Float */
code span.fu { color: #06287e; } /* Function */
code span.im { } /* Import */
code span.in { color: #60a0b0; font-weight: bold; font-style: italic; } /* Information */
code span.kw { color: #007020; font-weight: bold; } /* Keyword */
code span.op { color: #666666; } /* Operator */
code span.ot { color: #007020; } /* Other */
code span.pp { color: #bc7a00; } /* Preprocessor */
code span.sc { color: #4070a0; } /* SpecialChar */
code span.ss { color: #bb6688; } /* SpecialString */
code span.st { color: #4070a0; } /* String */
code span.va { color: #19177c; } /* Variable */
code span.vs { color: #4070a0; } /* VerbatimString */
code span.wa { color: #60a0b0; font-weight: bold; font-style: italic; } /* Warning */

/* A workaround for https://github.com/jgm/pandoc/issues/4278 */
a.sourceLine {
  pointer-events: auto;
}

</style>
<script>
// apply pandoc div.sourceCode style to pre.sourceCode instead
(function() {
  var sheets = document.styleSheets;
  for (var i = 0; i < sheets.length; i++) {
    if (sheets[i].ownerNode.dataset["origin"] !== "pandoc") continue;
    try { var rules = sheets[i].cssRules; } catch (e) { continue; }
    for (var j = 0; j < rules.length; j++) {
      var rule = rules[j];
      // check if there is a div.sourceCode rule
      if (rule.type !== rule.STYLE_RULE || rule.selectorText !== "div.sourceCode") continue;
      var style = rule.style.cssText;
      // check if color or background-color is set
      if (rule.style.color === '' && rule.style.backgroundColor === '') continue;
      // replace div.sourceCode by a pre.sourceCode rule
      sheets[i].deleteRule(j);
      sheets[i].insertRule('pre.sourceCode{' + style + '}', j);
    }
  }
})();
</script>



<style type="text/css">@font-face{font-family:'Open Sans';font-style:normal;font-weight:400;src:local('Open Sans'),local(OpenSans),url(data:application/font-woff;base64,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) format("woff")}@font-face{font-family:'Open Sans';font-style:normal;font-weight:700;src:local('Open Sans Bold'),local(OpenSans-Bold),url(data:application/font-woff;base64,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) format("woff")}*{box-sizing:border-box}body{padding:0;margin:0;font-family:"Open Sans","Helvetica Neue",Helvetica,Arial,sans-serif;font-size:16px;line-height:1.5;color:#606c71}a{color:#1e6bb8;text-decoration:none}a:hover{text-decoration:underline}.page-header{color:#fff;text-align:center;background-color:#159957;background-image:linear-gradient(120deg,#155799,#159957);padding:1.5rem 2rem}.project-name{margin-top:0;margin-bottom:.1rem;font-size:2rem}.project-tagline{margin-bottom:2rem;font-weight:400;opacity:.7;font-size:1.5rem}.project-author,.project-date{font-weight:400;opacity:.7;font-size:1.2rem}@media screen and (max-width: 42em){.page-header{padding:1rem}.project-name{font-size:1.75rem}.project-tagline{font-size:1.2rem}.project-author,.project-date{font-size:1rem}}.main-content:first-child{margin-top:0}.main-content img{max-width:100%}.main-content h1,.main-content h2,.main-content h3,.main-content h4,.main-content h5,.main-content h6{margin-top:2rem;margin-bottom:1rem;font-weight:400;color:#159957}.main-content p{margin-bottom:1em}.main-content code{padding:2px 4px;font-family:Consolas,"Liberation Mono",Menlo,Courier,monospace;color:#383e41;background-color:#f3f6fa;border-radius:.3rem}.main-content pre{padding:.8rem;margin-top:0;margin-bottom:1rem;font:1rem Consolas,"Liberation Mono",Menlo,Courier,monospace;color:#567482;word-wrap:normal;background-color:#f3f6fa;border:solid 1px #dce6f0;border-radius:.3rem;line-height:1.45;overflow:auto}.main-content pre> code{padding:0;margin:0;font-size:1rem;color:#567482;word-break:normal;white-space:pre;background:transparent;border:0}.main-content pre code,.main-content pre tt{display:inline;padding:0;line-height:inherit;word-wrap:normal;background-color:transparent;border:0}.main-content pre code:before,.main-content pre code:after,.main-content pre tt:before,.main-content pre tt:after{content:normal}.main-content ul,.main-content ol{margin-top:0}.main-content blockquote{padding:0 1rem;margin-left:0;font-size:1.2rem;color:#819198;border-left:.3rem solid #dce6f0}.main-content blockquote>:first-child{margin-top:0}.main-content blockquote>:last-child{margin-bottom:0}.main-content table{width:100%;overflow:auto;word-break:normal;word-break:keep-all;border-collapse:collapse;border-spacing:0;margin:1rem 0}.main-content table th{font-weight:700;background-color:#4CAF50;color:#fff}.main-content table th,.main-content table td{padding:.5rem 1rem;border-bottom:1px solid #e9ebec;text-align:left}.main-content table tr:nth-child(odd){background-color:#f2f2f2}.main-content dl{padding:0}.main-content dl dt{padding:0;margin-top:1rem;font-size:1rem;font-weight:700}.main-content dl dd{padding:0;margin-bottom:1rem}.main-content hr{margin:1rem 0;border:0;height:1px;background:#aaa;background-image:linear-gradient(to right,#eee,#aaa,#eee)}.main-content,.toc{max-width:64rem;padding:2rem 4rem;margin:0 auto;font-size:1.1rem}.toc{padding-bottom:0}.toc ul{margin-bottom:0}@media screen and (min-width: 42em) and (max-width: 64em){.toc{padding:2rem 2rem 0}.main-content{padding:2rem}}@media screen and (max-width: 42em){.toc{padding:2rem 1rem 0;font-size:1rem}.main-content{padding:2rem 1rem;font-size:1rem}.main-content pre,.main-content pre> code{font-size:.9rem}.main-content blockquote{font-size:1.1rem}}.site-footer{padding-top:2rem;margin-top:2rem;border-top:solid 1px #eff0f1;font-size:1rem}.site-footer-owner{display:block;font-weight:700}.site-footer-credits{color:#819198}
code span.kw { color: #a71d5d; font-weight: normal; } 
code span.dt { color: #795da3; } 
code span.dv { color: #0086b3; } 
code span.bn { color: #0086b3; } 
code span.fl { color: #0086b3; } 
code span.ch { color: #4070a0; } 
code span.st { color: #183691; } 
code span.co { color: #969896; font-style: italic; } 
code span.ot { color: #007020; } 
</style>





</head>

<body>




<section class="page-header">
<h1 class="title toc-ignore project-name">NBA Simulation</h1>
<h4 class="author project-author">Trevor Johnson</h4>
<h4 class="date project-date">3/11/2020</h4>
</section>



<section class="main-content">
<div id="start" class="section level1">
<h1>Start</h1>
<p>Simulating NBA games and predicting wins using Monte Carlo Simulation</p>
<p>Libraries needed</p>
<div class="sourceCode" id="cb1"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb1-1" data-line-number="1">libraries &lt;-<span class="st"> </span><span class="kw">c</span>(<span class="st">&quot;XML&quot;</span>, <span class="st">&quot;rvest&quot;</span>, <span class="st">&quot;dplyr&quot;</span>, <span class="st">&quot;data.table&quot;</span>, <span class="st">&quot;DT&quot;</span>, <span class="st">&quot;ggplot2&quot;</span>, <span class="st">&quot;patchwork&quot;</span>)</a>
<a class="sourceLine" id="cb1-2" data-line-number="2"><span class="kw">sapply</span>(libraries, require, <span class="dt">character.only =</span> T)</a></code></pre></div>
</div>
<div id="gathering-data" class="section level1">
<h1>Gathering Data</h1>
<p>Scrape real time data</p>
<div class="sourceCode" id="cb2"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb2-1" data-line-number="1"><span class="co"># https://www.basketball-reference.com/leagues/NBA_2020_totals.html</span></a>
<a class="sourceLine" id="cb2-2" data-line-number="2"><span class="co"># you can change the year to get historical stats</span></a>
<a class="sourceLine" id="cb2-3" data-line-number="3"></a>
<a class="sourceLine" id="cb2-4" data-line-number="4"><span class="co"># libraries</span></a>
<a class="sourceLine" id="cb2-5" data-line-number="5">libraries &lt;-<span class="st"> </span><span class="kw">c</span>(<span class="st">&quot;XML&quot;</span>, <span class="st">&quot;rvest&quot;</span>, <span class="st">&quot;dplyr&quot;</span>, <span class="st">&quot;data.table&quot;</span>)</a>
<a class="sourceLine" id="cb2-6" data-line-number="6"><span class="kw">sapply</span>(libraries, require, <span class="dt">character.only =</span> T)</a>
<a class="sourceLine" id="cb2-7" data-line-number="7"></a>
<a class="sourceLine" id="cb2-8" data-line-number="8"></a>
<a class="sourceLine" id="cb2-9" data-line-number="9"><span class="co"># Scraping the regular player data</span></a>
<a class="sourceLine" id="cb2-10" data-line-number="10">url &lt;-<span class="st"> &quot;https://www.basketball-reference.com/leagues/NBA_2020_totals.html&quot;</span></a>
<a class="sourceLine" id="cb2-11" data-line-number="11">css &lt;-<span class="st"> &quot;td&quot;</span> <span class="co"># other CSS options: &quot;#div_totals_stats&quot;, &quot;.right , .left , .center&quot;</span></a>
<a class="sourceLine" id="cb2-12" data-line-number="12">stats &lt;-<span class="st"> </span>rvest<span class="op">::</span><span class="kw">html_text</span>(rvest<span class="op">::</span><span class="kw">html_nodes</span>(xml2<span class="op">::</span><span class="kw">read_html</span>(url), css))</a>
<a class="sourceLine" id="cb2-13" data-line-number="13"></a>
<a class="sourceLine" id="cb2-14" data-line-number="14"><span class="co"># each df should have these 29 fields:</span></a>
<a class="sourceLine" id="cb2-15" data-line-number="15">colHeaders &lt;-<span class="st"> </span><span class="kw">c</span>(<span class="st">&quot;Player&quot;</span>, <span class="st">&quot;Pos&quot;</span>, <span class="st">&quot;Age&quot;</span>, <span class="st">&quot;Tm&quot;</span>, <span class="st">&quot;G&quot;</span>, <span class="st">&quot;GS&quot;</span>, <span class="st">&quot;MP&quot;</span>, <span class="st">&quot;FG&quot;</span>, <span class="st">&quot;FGA&quot;</span>, <span class="st">&quot;FG%&quot;</span>, <span class="st">&quot;3P&quot;</span>, <span class="st">&quot;3PA&quot;</span>, <span class="st">&quot;3P%&quot;</span>, <span class="st">&quot;2P&quot;</span>, <span class="st">&quot;2PA&quot;</span>, <span class="st">&quot;2P%&quot;</span>, <span class="st">&quot;eFG%&quot;</span>, <span class="st">&quot;FT&quot;</span>, <span class="st">&quot;FTA&quot;</span>, <span class="st">&quot;FT%&quot;</span>, <span class="st">&quot;ORB&quot;</span>, <span class="st">&quot;DRB&quot;</span>, <span class="st">&quot;TRB&quot;</span>, <span class="st">&quot;AST&quot;</span>, <span class="st">&quot;STL&quot;</span>, <span class="st">&quot;BLK&quot;</span>, <span class="st">&quot;TOV&quot;</span>, <span class="st">&quot;PF&quot;</span>, <span class="st">&quot;PTS&quot;</span>)</a>
<a class="sourceLine" id="cb2-16" data-line-number="16"></a>
<a class="sourceLine" id="cb2-17" data-line-number="17"><span class="co"># Every 29 elements in the large &quot;stats&quot; vector is a new row, so looping through to convert to data.frame format</span></a>
<a class="sourceLine" id="cb2-18" data-line-number="18">df &lt;-<span class="st"> </span><span class="kw">cbind</span>(stats[<span class="kw">seq</span>(<span class="dv">1</span>, <span class="dv">600</span><span class="op">*</span><span class="dv">29</span>, <span class="dt">by =</span> <span class="dv">29</span>)], stats[<span class="kw">seq</span>(<span class="dv">2</span>, <span class="dv">600</span><span class="op">*</span><span class="dv">29</span>, <span class="dt">by =</span> <span class="dv">29</span>)])</a>
<a class="sourceLine" id="cb2-19" data-line-number="19"><span class="cf">for</span>(i <span class="cf">in</span> <span class="dv">3</span><span class="op">:</span><span class="dv">29</span>){</a>
<a class="sourceLine" id="cb2-20" data-line-number="20">  df &lt;-<span class="st"> </span><span class="kw">cbind</span>(df, stats[<span class="kw">seq</span>(i, <span class="dv">600</span><span class="op">*</span><span class="dv">29</span>, <span class="dt">by =</span> <span class="dv">29</span>)])</a>
<a class="sourceLine" id="cb2-21" data-line-number="21">}</a>
<a class="sourceLine" id="cb2-22" data-line-number="22">df &lt;-<span class="st"> </span><span class="kw">data.frame</span>(df)</a>
<a class="sourceLine" id="cb2-23" data-line-number="23"><span class="kw">names</span>(df) &lt;-<span class="st"> </span>colHeaders</a>
<a class="sourceLine" id="cb2-24" data-line-number="24"></a>
<a class="sourceLine" id="cb2-25" data-line-number="25"><span class="co"># cleaning up the data.frame</span></a>
<a class="sourceLine" id="cb2-26" data-line-number="26">df &lt;-<span class="st"> </span>df[<span class="op">!</span><span class="kw">is.na</span>(df<span class="op">$</span>Player),] <span class="co"># deleting extra rows</span></a>
<a class="sourceLine" id="cb2-27" data-line-number="27"><span class="co"># changing column types</span></a>
<a class="sourceLine" id="cb2-28" data-line-number="28">df[colHeaders[<span class="dv">1</span><span class="op">:</span><span class="dv">29</span>]] &lt;-<span class="st"> </span><span class="kw">sapply</span>(df[colHeaders[<span class="dv">1</span><span class="op">:</span><span class="dv">29</span>]], as.character)</a>
<a class="sourceLine" id="cb2-29" data-line-number="29">df[df <span class="op">==</span><span class="st"> &quot;&quot;</span>] &lt;-<span class="st"> &quot;0&quot;</span> <span class="co"># replacing missing values w/ 0's</span></a>
<a class="sourceLine" id="cb2-30" data-line-number="30">df[colHeaders[<span class="kw">c</span>(<span class="dv">3</span>, <span class="dv">5</span><span class="op">:</span><span class="dv">29</span>)]] &lt;-<span class="st"> </span><span class="kw">sapply</span>(df[colHeaders[<span class="kw">c</span>(<span class="dv">3</span>, <span class="dv">5</span><span class="op">:</span><span class="dv">29</span>)]], as.numeric)</a>
<a class="sourceLine" id="cb2-31" data-line-number="31">totals &lt;-<span class="st"> </span>df</a>
<a class="sourceLine" id="cb2-32" data-line-number="32"></a>
<a class="sourceLine" id="cb2-33" data-line-number="33"></a>
<a class="sourceLine" id="cb2-34" data-line-number="34"></a>
<a class="sourceLine" id="cb2-35" data-line-number="35"><span class="co"># now scraping the advanced table</span></a>
<a class="sourceLine" id="cb2-36" data-line-number="36"><span class="co"># Scraping the data</span></a>
<a class="sourceLine" id="cb2-37" data-line-number="37">url &lt;-<span class="st"> &quot;https://www.basketball-reference.com/leagues/NBA_2020_advanced.html&quot;</span></a>
<a class="sourceLine" id="cb2-38" data-line-number="38">css &lt;-<span class="st"> &quot;td&quot;</span> <span class="co"># other CSS options: &quot;#div_totals_stats&quot;, &quot;.right , .left , .center&quot;</span></a>
<a class="sourceLine" id="cb2-39" data-line-number="39">stats &lt;-<span class="st"> </span>rvest<span class="op">::</span><span class="kw">html_text</span>(rvest<span class="op">::</span><span class="kw">html_nodes</span>(xml2<span class="op">::</span><span class="kw">read_html</span>(url), css))</a>
<a class="sourceLine" id="cb2-40" data-line-number="40"></a>
<a class="sourceLine" id="cb2-41" data-line-number="41"><span class="co"># there are 2 columns with blank data in the dataset</span></a>
<a class="sourceLine" id="cb2-42" data-line-number="42">colHeaders &lt;-<span class="st"> </span><span class="kw">c</span>(<span class="st">&quot;Player&quot;</span>, <span class="st">&quot;Pos&quot;</span>, <span class="st">&quot;Age&quot;</span>, <span class="st">&quot;Tm&quot;</span>, <span class="st">&quot;G&quot;</span>, <span class="st">'MP'</span>, <span class="st">&quot;PER&quot;</span>, <span class="st">&quot;TS%&quot;</span>, <span class="st">&quot;3PAr&quot;</span>, <span class="st">&quot;FTr&quot;</span>, <span class="st">&quot;ORB%&quot;</span>, <span class="st">&quot;DRB%&quot;</span>, <span class="st">&quot;TRB%&quot;</span>, <span class="st">&quot;AST%&quot;</span>, <span class="st">&quot;STL%&quot;</span>, <span class="st">&quot;BLK%&quot;</span>, <span class="st">&quot;TOV%&quot;</span>, <span class="st">&quot;USG%&quot;</span>, <span class="st">&quot;blank1&quot;</span>, <span class="st">&quot;OWS&quot;</span>, <span class="st">&quot;DWS&quot;</span>, <span class="st">&quot;WS&quot;</span>, <span class="st">&quot;WS/48&quot;</span>, <span class="st">&quot;blank2&quot;</span>, <span class="st">&quot;OBPM&quot;</span>, <span class="st">&quot;DBPM&quot;</span>, <span class="st">&quot;BPM&quot;</span>, <span class="st">&quot;VORP&quot;</span>)</a>
<a class="sourceLine" id="cb2-43" data-line-number="43"></a>
<a class="sourceLine" id="cb2-44" data-line-number="44"><span class="co"># new player every 28 records</span></a>
<a class="sourceLine" id="cb2-45" data-line-number="45">df &lt;-<span class="st"> </span><span class="kw">cbind</span>(stats[<span class="kw">seq</span>(<span class="dv">1</span>, <span class="dv">600</span><span class="op">*</span><span class="dv">28</span>, <span class="dt">by =</span> <span class="dv">28</span>)], stats[<span class="kw">seq</span>(<span class="dv">2</span>, <span class="dv">600</span><span class="op">*</span><span class="dv">28</span>, <span class="dt">by =</span> <span class="dv">28</span>)])</a>
<a class="sourceLine" id="cb2-46" data-line-number="46"><span class="cf">for</span>(i <span class="cf">in</span> <span class="dv">3</span><span class="op">:</span><span class="dv">28</span>){</a>
<a class="sourceLine" id="cb2-47" data-line-number="47">  df &lt;-<span class="st"> </span><span class="kw">cbind</span>(df, stats[<span class="kw">seq</span>(i, <span class="dv">600</span><span class="op">*</span><span class="dv">28</span>, <span class="dt">by =</span> <span class="dv">28</span>)])</a>
<a class="sourceLine" id="cb2-48" data-line-number="48">}</a>
<a class="sourceLine" id="cb2-49" data-line-number="49">df &lt;-<span class="st"> </span><span class="kw">data.frame</span>(df)</a>
<a class="sourceLine" id="cb2-50" data-line-number="50"><span class="kw">names</span>(df) &lt;-<span class="st"> </span>colHeaders</a>
<a class="sourceLine" id="cb2-51" data-line-number="51"></a>
<a class="sourceLine" id="cb2-52" data-line-number="52"><span class="co"># data clean</span></a>
<a class="sourceLine" id="cb2-53" data-line-number="53">df &lt;-<span class="st"> </span>df[<span class="op">!</span><span class="kw">is.na</span>(df<span class="op">$</span>Player),] <span class="co"># deleting extra rows</span></a>
<a class="sourceLine" id="cb2-54" data-line-number="54">df[colHeaders[<span class="dv">1</span><span class="op">:</span><span class="dv">28</span>]] &lt;-<span class="st"> </span><span class="kw">sapply</span>(df[colHeaders[<span class="kw">c</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">28</span>)]], as.character)</a>
<a class="sourceLine" id="cb2-55" data-line-number="55">df[df <span class="op">==</span><span class="st"> &quot;&quot;</span>] &lt;-<span class="st"> &quot;0&quot;</span> <span class="co"># replacing missing values w/ 0's</span></a>
<a class="sourceLine" id="cb2-56" data-line-number="56">df[colHeaders[<span class="kw">c</span>(<span class="dv">3</span>, <span class="dv">5</span><span class="op">:</span><span class="dv">28</span>)]] &lt;-<span class="st"> </span><span class="kw">sapply</span>(df[colHeaders[<span class="kw">c</span>(<span class="dv">3</span>, <span class="dv">5</span><span class="op">:</span><span class="dv">28</span>)]], as.numeric)</a>
<a class="sourceLine" id="cb2-57" data-line-number="57">advanced &lt;-<span class="st"> </span>df</a>
<a class="sourceLine" id="cb2-58" data-line-number="58"></a>
<a class="sourceLine" id="cb2-59" data-line-number="59"><span class="co">#----------------------------------------------------------------------------------------------------</span></a>
<a class="sourceLine" id="cb2-60" data-line-number="60"><span class="co"># joining the two tables</span></a>
<a class="sourceLine" id="cb2-61" data-line-number="61">players_all &lt;-<span class="st"> </span><span class="kw">left_join</span>(totals, <span class="kw">select</span>(advanced, Player, Tm, PER, <span class="st">`</span><span class="dt">TS%</span><span class="st">`</span>, DWS), <span class="dt">by =</span> <span class="kw">c</span>(<span class="st">&quot;Player&quot;</span>, <span class="st">&quot;Tm&quot;</span>))</a>
<a class="sourceLine" id="cb2-62" data-line-number="62"></a>
<a class="sourceLine" id="cb2-63" data-line-number="63"><span class="co">#Saving the output for reproducable results</span></a>
<a class="sourceLine" id="cb2-64" data-line-number="64"><span class="co">#write.csv(players_all, &quot;/Users/tj/Documents/Data/NBA/PlayerStats_2020.03.10.csv&quot;, row.names = FALSE)</span></a></code></pre></div>
</div>
<div id="data-cleaning" class="section level1">
<h1>Data Cleaning</h1>
<div class="sourceCode" id="cb3"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb3-1" data-line-number="1">players_all &lt;-<span class="st"> </span><span class="kw">fread</span>(<span class="st">&quot;/Users/tj/Documents/Data/NBA/PlayerStats_2020.03.10.csv&quot;</span>)</a>
<a class="sourceLine" id="cb3-2" data-line-number="2">players &lt;-<span class="st"> </span>players_all <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">select</span>(Player, Tm, <span class="st">`</span><span class="dt">3PA</span><span class="st">`</span>, <span class="st">`</span><span class="dt">3P%</span><span class="st">`</span>, <span class="st">`</span><span class="dt">2PA</span><span class="st">`</span>, <span class="st">`</span><span class="dt">2P%</span><span class="st">`</span>, FTA, <span class="st">`</span><span class="dt">FT%</span><span class="st">`</span>, TOV, ORB, DRB, DWS, G, MP) <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">data.table</span>()</a>
<a class="sourceLine" id="cb3-3" data-line-number="3"><span class="co"># getting each players probability of playing within each team</span></a>
<a class="sourceLine" id="cb3-4" data-line-number="4">players<span class="op">$</span>DWS_zscore &lt;-<span class="st"> </span>(players<span class="op">$</span>DWS <span class="op">-</span><span class="st"> </span><span class="kw">mean</span>(players<span class="op">$</span>DWS))<span class="op">/</span><span class="kw">sd</span>(players<span class="op">$</span>DWS)</a>
<a class="sourceLine" id="cb3-5" data-line-number="5">players<span class="op">$</span>DWS_scaled &lt;-<span class="st"> </span>(players<span class="op">$</span>DWS_zscore <span class="op">-</span><span class="st"> </span><span class="kw">min</span>(players<span class="op">$</span>DWS_zscore)) <span class="op">/</span><span class="st"> </span>(<span class="kw">max</span>(players<span class="op">$</span>DWS_zscore) <span class="op">-</span><span class="st"> </span><span class="kw">min</span>(players<span class="op">$</span>DWS_zscore))</a>
<a class="sourceLine" id="cb3-6" data-line-number="6">players<span class="op">$</span>DWS_scaled &lt;-<span class="st"> </span>players<span class="op">$</span>DWS_scaled <span class="op">-</span><span class="st"> </span><span class="kw">mean</span>(players<span class="op">$</span>DWS_scaled)</a>
<a class="sourceLine" id="cb3-7" data-line-number="7">players<span class="op">$</span>DWS_zscore &lt;-<span class="st"> </span><span class="ot">NULL</span></a></code></pre></div>
</div>
<div id="possession-function" class="section level1">
<h1>Possession Function</h1>
<div class="sourceCode" id="cb4"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb4-1" data-line-number="1"><span class="co"># possession function</span></a>
<a class="sourceLine" id="cb4-2" data-line-number="2"><span class="co">## read this as team A is on offense, team B is on defense</span></a>
<a class="sourceLine" id="cb4-3" data-line-number="3"></a>
<a class="sourceLine" id="cb4-4" data-line-number="4"><span class="co"># need to initialize this outside of the function to reset, then loop the possession function</span></a>
<a class="sourceLine" id="cb4-5" data-line-number="5">playByPlay &lt;-<span class="st"> </span><span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Player&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Event&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="kw">double</span>(), <span class="dt">stringsAsFactors =</span> F) </a>
<a class="sourceLine" id="cb4-6" data-line-number="6"></a>
<a class="sourceLine" id="cb4-7" data-line-number="7"><span class="co"># team_A = team w/ the ball</span></a>
<a class="sourceLine" id="cb4-8" data-line-number="8"><span class="co"># team_B = team defending</span></a>
<a class="sourceLine" id="cb4-9" data-line-number="9"><span class="co"># printPlayByPlay = bool, if you run the function once, it's nice to set this to TRUE</span></a>
<a class="sourceLine" id="cb4-10" data-line-number="10"><span class="co"># def_multiplier = some number, 0 means defense has no impact, 1 means a regular impact, 2 means 2x, 3 = 3x, etc.</span></a>
<a class="sourceLine" id="cb4-11" data-line-number="11"></a>
<a class="sourceLine" id="cb4-12" data-line-number="12">possession &lt;-<span class="st"> </span><span class="cf">function</span>(team_A, team_B, <span class="dt">printPlayByPlay =</span> <span class="ot">TRUE</span>, <span class="dt">def_multiplier =</span> <span class="dv">1</span>){</a>
<a class="sourceLine" id="cb4-13" data-line-number="13">  </a>
<a class="sourceLine" id="cb4-14" data-line-number="14">  team_A1 =<span class="st"> </span>players[Tm <span class="op">==</span><span class="st"> </span>team_A]</a>
<a class="sourceLine" id="cb4-15" data-line-number="15">  team_B1 =<span class="st"> </span>players[Tm <span class="op">==</span><span class="st"> </span>team_B]</a>
<a class="sourceLine" id="cb4-16" data-line-number="16">  </a>
<a class="sourceLine" id="cb4-17" data-line-number="17">  <span class="co"># using total season minutes played (MP) as play probability</span></a>
<a class="sourceLine" id="cb4-18" data-line-number="18">  team_A5 =<span class="st"> </span>team_A1[Player <span class="op">%in%</span><span class="st"> </span><span class="kw">sample</span>(team_A1<span class="op">$</span>Player, <span class="dt">size =</span> <span class="dv">5</span>, <span class="dt">replace =</span> <span class="ot">FALSE</span>, <span class="dt">prob =</span> team_A1<span class="op">$</span>MP)]</a>
<a class="sourceLine" id="cb4-19" data-line-number="19">  team_B5 =<span class="st"> </span>team_B1[Player <span class="op">%in%</span><span class="st"> </span><span class="kw">sample</span>(team_B1<span class="op">$</span>Player, <span class="dt">size =</span> <span class="dv">5</span>, <span class="dt">replace =</span> <span class="ot">FALSE</span>, <span class="dt">prob =</span> team_B1<span class="op">$</span>MP)]</a>
<a class="sourceLine" id="cb4-20" data-line-number="20">  </a>
<a class="sourceLine" id="cb4-21" data-line-number="21">  </a>
<a class="sourceLine" id="cb4-22" data-line-number="22">  <span class="co"># action</span></a>
<a class="sourceLine" id="cb4-23" data-line-number="23">  possible_actions &lt;-<span class="st"> </span><span class="kw">c</span>(<span class="kw">paste0</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">5</span>, <span class="st">&quot;_2P%&quot;</span>), <span class="kw">paste0</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">5</span>, <span class="st">&quot;_3P%&quot;</span>), <span class="kw">paste0</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">5</span>, <span class="st">&quot;_FT%&quot;</span>), <span class="kw">paste0</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">5</span>, <span class="st">&quot;_TOV&quot;</span>))</a>
<a class="sourceLine" id="cb4-24" data-line-number="24">  prob_weights &lt;-<span class="st"> </span><span class="kw">c</span>(team_A5<span class="op">$</span><span class="st">`</span><span class="dt">2PA</span><span class="st">`</span>, team_A5<span class="op">$</span><span class="st">`</span><span class="dt">3PA</span><span class="st">`</span>, team_A5<span class="op">$</span>FTA, team_A5<span class="op">$</span>TOV)</a>
<a class="sourceLine" id="cb4-25" data-line-number="25">  </a>
<a class="sourceLine" id="cb4-26" data-line-number="26">  rebound_tbl &lt;-<span class="st"> </span><span class="kw">data.table</span>(<span class="dt">Player =</span> <span class="kw">c</span>(team_A5<span class="op">$</span>Player, team_B5<span class="op">$</span>Player), <span class="dt">Tm =</span> <span class="kw">c</span>(team_A5<span class="op">$</span>Tm, team_B5<span class="op">$</span>Tm), <span class="dt">Rebounds =</span> <span class="kw">c</span>(team_A5<span class="op">$</span>ORB, team_B5<span class="op">$</span>DRB))</a>
<a class="sourceLine" id="cb4-27" data-line-number="27">  </a>
<a class="sourceLine" id="cb4-28" data-line-number="28">  <span class="co"># away team's defensive impact</span></a>
<a class="sourceLine" id="cb4-29" data-line-number="29">  defImpact &lt;-<span class="st"> </span><span class="kw">mean</span>(team_B5<span class="op">$</span>DWS_scaled)<span class="op">*-</span><span class="dv">1</span><span class="op">*</span>def_multiplier</a>
<a class="sourceLine" id="cb4-30" data-line-number="30">  </a>
<a class="sourceLine" id="cb4-31" data-line-number="31">  <span class="co"># what happens? </span></a>
<a class="sourceLine" id="cb4-32" data-line-number="32">  <span class="co"># Initialize these variables to begin, and they will eventually reset</span></a>
<a class="sourceLine" id="cb4-33" data-line-number="33">  <span class="co"># event &lt;- &quot;miss&quot;</span></a>
<a class="sourceLine" id="cb4-34" data-line-number="34">  rebound &lt;-<span class="st"> &quot;homeRebound&quot;</span></a>
<a class="sourceLine" id="cb4-35" data-line-number="35">  </a>
<a class="sourceLine" id="cb4-36" data-line-number="36">  <span class="co"># the actual play</span></a>
<a class="sourceLine" id="cb4-37" data-line-number="37">  <span class="cf">while</span>(rebound <span class="op">==</span><span class="st"> &quot;homeRebound&quot;</span>){</a>
<a class="sourceLine" id="cb4-38" data-line-number="38">    </a>
<a class="sourceLine" id="cb4-39" data-line-number="39">    theAction &lt;-<span class="st"> </span><span class="kw">sample</span>(possible_actions, <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> prob_weights)</a>
<a class="sourceLine" id="cb4-40" data-line-number="40">    theAction2 &lt;-<span class="st"> </span><span class="kw">c</span>(<span class="kw">substr</span>(theAction, <span class="dv">1</span>, <span class="dv">1</span>), <span class="kw">substr</span>(theAction, <span class="dv">3</span>, <span class="kw">nchar</span>(theAction))) </a>
<a class="sourceLine" id="cb4-41" data-line-number="41">    prb &lt;-<span class="st"> </span>(team_A5[<span class="kw">as.numeric</span>(theAction2[<span class="dv">1</span>]),] <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">select</span>(theAction2[<span class="dv">2</span>]))<span class="op">*</span>(<span class="dv">1</span><span class="op">+</span>defImpact) <span class="co">#def impact happens here, only impacts shooting, is already inherent in rebound counts</span></a>
<a class="sourceLine" id="cb4-42" data-line-number="42">    </a>
<a class="sourceLine" id="cb4-43" data-line-number="43">    <span class="cf">if</span>(theAction2[<span class="dv">2</span>] <span class="op">==</span><span class="st"> &quot;TOV&quot;</span>){</a>
<a class="sourceLine" id="cb4-44" data-line-number="44">      rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span> <span class="co"># to end the while loop</span></a>
<a class="sourceLine" id="cb4-45" data-line-number="45">      <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>team_A, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>team_A5[<span class="kw">as.numeric</span>(theAction2[<span class="dv">1</span>]),]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;Turnover&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="dv">0</span>, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) <span class="co"># save event</span></a>
<a class="sourceLine" id="cb4-46" data-line-number="46">      </a>
<a class="sourceLine" id="cb4-47" data-line-number="47">    }<span class="cf">else</span> <span class="cf">if</span>(theAction2[<span class="dv">2</span>] <span class="op">==</span><span class="st"> &quot;2P%&quot;</span>){</a>
<a class="sourceLine" id="cb4-48" data-line-number="48">      </a>
<a class="sourceLine" id="cb4-49" data-line-number="49">      pt &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="kw">c</span>(<span class="dv">0</span>, <span class="dv">2</span>), <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> <span class="kw">c</span>(<span class="dv">1</span><span class="op">-</span>prb, prb))</a>
<a class="sourceLine" id="cb4-50" data-line-number="50">      <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>team_A, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>team_A5[<span class="kw">as.numeric</span>(theAction2[<span class="dv">1</span>]),]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;2pt&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span>pt, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) <span class="co"># save event</span></a>
<a class="sourceLine" id="cb4-51" data-line-number="51">      </a>
<a class="sourceLine" id="cb4-52" data-line-number="52">      <span class="cf">if</span>(pt <span class="op">==</span><span class="st"> </span><span class="dv">0</span>){</a>
<a class="sourceLine" id="cb4-53" data-line-number="53">        </a>
<a class="sourceLine" id="cb4-54" data-line-number="54">        rebound_ind &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">10</span>, <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> rebound_tbl<span class="op">$</span>Rebounds)</a>
<a class="sourceLine" id="cb4-55" data-line-number="55">        <span class="cf">if</span>(rebound_ind <span class="op">&lt;=</span><span class="st"> </span><span class="dv">5</span>){rebound &lt;-<span class="st"> &quot;homeRebound&quot;</span>}<span class="cf">else</span>{rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span>}</a>
<a class="sourceLine" id="cb4-56" data-line-number="56">        <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>rebound_tbl[rebound_ind,]<span class="op">$</span>Tm, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>rebound_tbl[rebound_ind,]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;Rebound&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="dv">0</span>, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) </a>
<a class="sourceLine" id="cb4-57" data-line-number="57">        </a>
<a class="sourceLine" id="cb4-58" data-line-number="58">      }<span class="cf">else</span>{</a>
<a class="sourceLine" id="cb4-59" data-line-number="59">        rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span> <span class="co"># to end while loop</span></a>
<a class="sourceLine" id="cb4-60" data-line-number="60">      }</a>
<a class="sourceLine" id="cb4-61" data-line-number="61">      </a>
<a class="sourceLine" id="cb4-62" data-line-number="62">    }<span class="cf">else</span> <span class="cf">if</span>(theAction2[<span class="dv">2</span>] <span class="op">==</span><span class="st"> &quot;3P%&quot;</span>){</a>
<a class="sourceLine" id="cb4-63" data-line-number="63">      </a>
<a class="sourceLine" id="cb4-64" data-line-number="64">      pt &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="kw">c</span>(<span class="dv">0</span>, <span class="dv">3</span>), <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> <span class="kw">c</span>(<span class="dv">1</span><span class="op">-</span>prb, prb))</a>
<a class="sourceLine" id="cb4-65" data-line-number="65">      <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>team_A, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>team_A5[<span class="kw">as.numeric</span>(theAction2[<span class="dv">1</span>]),]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;3pt&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span>pt, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) <span class="co"># save event</span></a>
<a class="sourceLine" id="cb4-66" data-line-number="66">      <span class="cf">if</span>(pt <span class="op">==</span><span class="st"> </span><span class="dv">0</span>){</a>
<a class="sourceLine" id="cb4-67" data-line-number="67">        </a>
<a class="sourceLine" id="cb4-68" data-line-number="68">        rebound_ind &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">10</span>, <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> rebound_tbl<span class="op">$</span>Rebounds)</a>
<a class="sourceLine" id="cb4-69" data-line-number="69">        <span class="cf">if</span>(rebound_ind <span class="op">&lt;=</span><span class="st"> </span><span class="dv">5</span>){rebound &lt;-<span class="st"> &quot;homeRebound&quot;</span>}<span class="cf">else</span>{rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span>}</a>
<a class="sourceLine" id="cb4-70" data-line-number="70">        <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>rebound_tbl[rebound_ind,]<span class="op">$</span>Tm, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>rebound_tbl[rebound_ind,]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;Rebound&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="dv">0</span>, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) </a>
<a class="sourceLine" id="cb4-71" data-line-number="71">        </a>
<a class="sourceLine" id="cb4-72" data-line-number="72">      }<span class="cf">else</span>{</a>
<a class="sourceLine" id="cb4-73" data-line-number="73">        rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span></a>
<a class="sourceLine" id="cb4-74" data-line-number="74">      }</a>
<a class="sourceLine" id="cb4-75" data-line-number="75">      </a>
<a class="sourceLine" id="cb4-76" data-line-number="76">    }<span class="cf">else</span> <span class="cf">if</span>(theAction2[<span class="dv">2</span>] <span class="op">==</span><span class="st"> &quot;FT%&quot;</span>){</a>
<a class="sourceLine" id="cb4-77" data-line-number="77">      </a>
<a class="sourceLine" id="cb4-78" data-line-number="78">      pt &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="kw">c</span>(<span class="dv">0</span>, <span class="dv">1</span>), <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> <span class="kw">c</span>(<span class="dv">1</span><span class="op">-</span>prb, prb))</a>
<a class="sourceLine" id="cb4-79" data-line-number="79">      <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>team_A, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>team_A5[<span class="kw">as.numeric</span>(theAction2[<span class="dv">1</span>]),]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;FT&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span>pt, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) <span class="co"># save event</span></a>
<a class="sourceLine" id="cb4-80" data-line-number="80">      </a>
<a class="sourceLine" id="cb4-81" data-line-number="81">      pt &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="kw">c</span>(<span class="dv">0</span>, <span class="dv">1</span>), <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> <span class="kw">c</span>(<span class="dv">1</span><span class="op">-</span>prb, prb))</a>
<a class="sourceLine" id="cb4-82" data-line-number="82">      <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>team_A, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>team_A5[<span class="kw">as.numeric</span>(theAction2[<span class="dv">1</span>]),]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;FT&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span>pt, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) <span class="co"># save event</span></a>
<a class="sourceLine" id="cb4-83" data-line-number="83">      <span class="cf">if</span>(pt <span class="op">==</span><span class="st"> </span><span class="dv">0</span>){</a>
<a class="sourceLine" id="cb4-84" data-line-number="84">        </a>
<a class="sourceLine" id="cb4-85" data-line-number="85">        rebound_ind &lt;-<span class="st"> </span><span class="kw">sample</span>(<span class="dv">1</span><span class="op">:</span><span class="dv">10</span>, <span class="dt">size =</span> <span class="dv">1</span>, <span class="dt">prob =</span> rebound_tbl<span class="op">$</span>Rebounds)</a>
<a class="sourceLine" id="cb4-86" data-line-number="86">        <span class="cf">if</span>(rebound_ind <span class="op">&lt;=</span><span class="st"> </span><span class="dv">5</span>){rebound &lt;-<span class="st"> &quot;homeRebound&quot;</span>}<span class="cf">else</span>{rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span>}</a>
<a class="sourceLine" id="cb4-87" data-line-number="87">        <span class="kw">assign</span>(<span class="st">&quot;playByPlay&quot;</span>, <span class="kw">rbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span>rebound_tbl[rebound_ind,]<span class="op">$</span>Tm, <span class="st">&quot;Player&quot;</span> =<span class="st"> </span>rebound_tbl[rebound_ind,]<span class="op">$</span>Player, <span class="st">&quot;Event&quot;</span> =<span class="st"> &quot;Rebound&quot;</span>, <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="dv">0</span>, <span class="dt">stringsAsFactors =</span> F)), <span class="dt">envir =</span> <span class="kw">parent.frame</span>()) </a>
<a class="sourceLine" id="cb4-88" data-line-number="88">        </a>
<a class="sourceLine" id="cb4-89" data-line-number="89">      }<span class="cf">else</span>{</a>
<a class="sourceLine" id="cb4-90" data-line-number="90">        rebound &lt;-<span class="st"> &quot;awayRebound&quot;</span> <span class="co"># tto end while loop</span></a>
<a class="sourceLine" id="cb4-91" data-line-number="91">      }</a>
<a class="sourceLine" id="cb4-92" data-line-number="92">    }</a>
<a class="sourceLine" id="cb4-93" data-line-number="93">  }<span class="co"># end while</span></a>
<a class="sourceLine" id="cb4-94" data-line-number="94">  </a>
<a class="sourceLine" id="cb4-95" data-line-number="95">  <span class="cf">if</span>(printPlayByPlay <span class="op">==</span><span class="st"> </span><span class="ot">TRUE</span>){<span class="kw">print</span>(playByPlay)}</a>
<a class="sourceLine" id="cb4-96" data-line-number="96">  </a>
<a class="sourceLine" id="cb4-97" data-line-number="97">}</a></code></pre></div>
<p>Testing it out</p>
<div class="sourceCode" id="cb5"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb5-1" data-line-number="1"><span class="co"># testing it out</span></a>
<a class="sourceLine" id="cb5-2" data-line-number="2"><span class="kw">set.seed</span>(<span class="dv">123</span>)</a>
<a class="sourceLine" id="cb5-3" data-line-number="3">playByPlay &lt;-<span class="st"> </span><span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Player&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Event&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="kw">double</span>(), <span class="dt">stringsAsFactors =</span> F) </a>
<a class="sourceLine" id="cb5-4" data-line-number="4"><span class="kw">possession</span>(<span class="dt">team_A =</span> <span class="st">&quot;UTA&quot;</span>, <span class="dt">team_B =</span> <span class="st">&quot;TOR&quot;</span>, <span class="dt">printPlayByPlay =</span> F, <span class="dt">def_multiplier =</span> <span class="dv">1</span>)</a>
<a class="sourceLine" id="cb5-5" data-line-number="5"><span class="kw">possession</span>(<span class="dt">team_A =</span> <span class="st">&quot;TOR&quot;</span>, <span class="dt">team_B =</span> <span class="st">&quot;UTA&quot;</span>, <span class="dt">printPlayByPlay =</span> T, <span class="dt">def_multiplier =</span> <span class="dv">1</span>)</a></code></pre></div>
<pre><code>##   Team        Player    Event Points
## 1  UTA  Tony Bradley       FT      0
## 2  UTA  Tony Bradley       FT      1
## 3  TOR Norman Powell Turnover      0</code></pre>
<div class="sourceCode" id="cb7"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb7-1" data-line-number="1"><span class="co"># looping it for 1 game</span></a>
<a class="sourceLine" id="cb7-2" data-line-number="2">playByPlay &lt;-<span class="st"> </span><span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Player&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Event&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="kw">double</span>(), <span class="dt">stringsAsFactors =</span> F) </a>
<a class="sourceLine" id="cb7-3" data-line-number="3"><span class="cf">for</span>(i <span class="cf">in</span> <span class="dv">1</span><span class="op">:</span><span class="dv">100</span>){</a>
<a class="sourceLine" id="cb7-4" data-line-number="4">  <span class="kw">possession</span>(<span class="dt">team_A =</span> <span class="st">&quot;UTA&quot;</span>, <span class="dt">team_B =</span> <span class="st">&quot;TOR&quot;</span>, <span class="dt">printPlayByPlay =</span> F, <span class="dt">def_multiplier =</span> <span class="dv">1</span>)</a>
<a class="sourceLine" id="cb7-5" data-line-number="5">  <span class="kw">possession</span>(<span class="dt">team_A =</span> <span class="st">&quot;TOR&quot;</span>, <span class="dt">team_B =</span> <span class="st">&quot;UTA&quot;</span>, <span class="dt">printPlayByPlay =</span> F, <span class="dt">def_multiplier =</span> <span class="dv">1</span>)</a>
<a class="sourceLine" id="cb7-6" data-line-number="6">}</a>
<a class="sourceLine" id="cb7-7" data-line-number="7"><span class="kw">datatable</span>(playByPlay)</a></code></pre></div>
<div id="htmlwidget-5cc2be19f6247c90eed1" style="width:100%;height:auto;" class="datatables html-widget"></div>
<script type="application/json" data-for="htmlwidget-5cc2be19f6247c90eed1">{"x":{"filter":"none","data":[["1","2","3","4","5","6","7","8","9","10","11","12","13","14","15","16","17","18","19","20","21","22","23","24","25","26","27","28","29","30","31","32","33","34","35","36","37","38","39","40","41","42","43","44","45","46","47","48","49","50","51","52","53","54","55","56","57","58","59","60","61","62","63","64","65","66","67","68","69","70","71","72","73","74","75","76","77","78","79","80","81","82","83","84","85","86","87","88","89","90","91","92","93","94","95","96","97","98","99","100","101","102","103","104","105","106","107","108","109","110","111","112","113","114","115","116","117","118","119","120","121","122","123","124","125","126","127","128","129","130","131","132","133","134","135","136","137","138","139","140","141","142","143","144","145","146","147","148","149","150","151","152","153","154","155","156","157","158","159","160","161","162","163","164","165","166","167","168","169","170","171","172","173","174","175","176","177","178","179","180","181","182","183","184","185","186","187","188","189","190","191","192","193","194","195","196","197","198","199","200","201","202","203","204","205","206","207","208","209","210","211","212","213","214","215","216","217","218","219","220","221","222","223","224","225","226","227","228","229","230","231","232","233","234","235","236","237","238","239","240","241","242","243","244","245","246","247","248","249","250","251","252","253","254","255","256","257","258","259","260","261","262","263","264","265","266","267","268","269","270","271","272","273","274","275","276","277","278","279","280","281","282","283","284","285","286","287","288","289","290","291","292","293","294","295","296","297","298","299","300","301","302","303","304","305","306","307","308","309","310","311","312","313","314","315","316","317","318","319","320","321","322","323","324","325","326","327","328","329","330","331","332","333","334","335","336","337","338","339","340","341","342","343","344","345","346","347","348","349","350","351","352","353","354","355","356","357","358"],["UTA","TOR","UTA","UTA","TOR","TOR","TOR","UTA","TOR","UTA","UTA","TOR","UTA","UTA","TOR","TOR","UTA","UTA","UTA","UTA","TOR","TOR","TOR","TOR","UTA","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","TOR","UTA","TOR","UTA","UTA","UTA","UTA","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","UTA","UTA","UTA","UTA","TOR","TOR","TOR","TOR","UTA","UTA","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","TOR","UTA","UTA","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","UTA","UTA","TOR","UTA","UTA","UTA","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","TOR","UTA","TOR","TOR","UTA","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","UTA","TOR","TOR","UTA","UTA","UTA","TOR","TOR","UTA","TOR","UTA","TOR","UTA","UTA","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","TOR","UTA","TOR","TOR","TOR","TOR","UTA","TOR","UTA","UTA","TOR","UTA","TOR","TOR","UTA","TOR","UTA","TOR","UTA","UTA","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","UTA","TOR","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","UTA","UTA","TOR","UTA","TOR","UTA","UTA","TOR","UTA","TOR","TOR","TOR","UTA","TOR","UTA","TOR","TOR","TOR","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","UTA","UTA","TOR","UTA","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","UTA","TOR","TOR","TOR","UTA","UTA","UTA","UTA","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","UTA","TOR","UTA","UTA","TOR","TOR","UTA","TOR","UTA","UTA","UTA","UTA","TOR","TOR","TOR","UTA","UTA","TOR","TOR","TOR","UTA","TOR","TOR","UTA","UTA","TOR","TOR","UTA","TOR","TOR","UTA","UTA","TOR","UTA","UTA","UTA","TOR","UTA","UTA","UTA","TOR","TOR","TOR","UTA","UTA","TOR","UTA","UTA","TOR","TOR","UTA","UTA","TOR","UTA","TOR","TOR","TOR","TOR","UTA","UTA","TOR","TOR","UTA"],["Rudy Gobert","Kyle Lowry","Rudy Gobert","Donovan Mitchell","Serge Ibaka","Serge Ibaka","Serge Ibaka","Donovan Mitchell","Kyle Lowry","Donovan Mitchell","Mike Conley","Pascal Siakam","Royce O'Neale","Donovan Mitchell","Norman Powell","Pascal Siakam","Bojan Bogdanović","Donovan Mitchell","Rudy Gobert","Bojan Bogdanović","OG Anunoby","Pascal Siakam","OG Anunoby","Rondae Hollis-Jefferson","Bojan Bogdanović","Rondae Hollis-Jefferson","Jordan Clarkson","Bojan Bogdanović","Rondae Hollis-Jefferson","Matt Thomas","Mike Conley","Marc Gasol","Pascal Siakam","Joe Ingles","Serge Ibaka","Pascal Siakam","Pascal Siakam","Donovan Mitchell","OG Anunoby","Royce O'Neale","Bojan Bogdanović","Donovan Mitchell","Donovan Mitchell","Fred VanVleet","Bojan Bogdanović","Terence Davis","OG Anunoby","Donovan Mitchell","Donovan Mitchell","OG Anunoby","Fred VanVleet","Donovan Mitchell","Rudy Gobert","Donovan Mitchell","Jeff Green","Jeff Green","Jeff Green","Marc Gasol","Kyle Lowry","Pascal Siakam","Fred VanVleet","Rudy Gobert","Royce O'Neale","Rondae Hollis-Jefferson","Donovan Mitchell","Fred VanVleet","OG Anunoby","Bojan Bogdanović","Marc Gasol","Marc Gasol","Donovan Mitchell","Pascal Siakam","Pascal Siakam","Royce O'Neale","Royce O'Neale","Chris Boucher","Pascal Siakam","Rudy Gobert","Pascal Siakam","Pascal Siakam","Joe Ingles","Fred VanVleet","Pascal Siakam","Pascal Siakam","Rudy Gobert","Donovan Mitchell","Serge Ibaka","Emmanuel Mudiay","Serge Ibaka","Rondae Hollis-Jefferson","Donovan Mitchell","Chris Boucher","Pascal Siakam","Rudy Gobert","Fred VanVleet","Tony Bradley","Donovan Mitchell","Pascal Siakam","Joe Ingles","Rudy Gobert","Rudy Gobert","Chris Boucher","Rudy Gobert","Kyle Lowry","Kyle Lowry","Joe Ingles","Marc Gasol","Patrick McCaw","Bojan Bogdanović","Pascal Siakam","Norman Powell","Norman Powell","Donovan Mitchell","Norman Powell","Rondae Hollis-Jefferson","Royce O'Neale","Pascal Siakam","Rudy Gobert","Donovan Mitchell","Kyle Lowry","Kyle Lowry","Georges Niang","Fred VanVleet","Fred VanVleet","Donovan Mitchell","Rudy Gobert","Rudy Gobert","Patrick McCaw","Patrick McCaw","Donovan Mitchell","Joe Ingles","Mike Conley","Kyle Lowry","Pascal Siakam","Emmanuel Mudiay","Norman Powell","Donovan Mitchell","Norman Powell","Rudy Gobert","Rudy Gobert","Rudy Gobert","Pascal Siakam","Fred VanVleet","Joe Ingles","Donovan Mitchell","Rondae Hollis-Jefferson","Serge Ibaka","Joe Ingles","Fred VanVleet","OG Anunoby","Emmanuel Mudiay","Emmanuel Mudiay","Serge Ibaka","Serge Ibaka","Rudy Gobert","Rudy Gobert","Pascal Siakam","Pascal Siakam","Donovan Mitchell","Pascal Siakam","Kyle Lowry","Kyle Lowry","Donovan Mitchell","Chris Boucher","Terence Davis","Mike Conley","Malcolm Miller","Pascal Siakam","Bojan Bogdanović","Pascal Siakam","Pascal Siakam","Mike Conley","Rudy Gobert","Pascal Siakam","Terence Davis","Bojan Bogdanović","Fred VanVleet","Fred VanVleet","Donovan Mitchell","Marc Gasol","OG Anunoby","Donovan Mitchell","Jordan Clarkson","Norman Powell","Pascal Siakam","Pascal Siakam","Joe Ingles","OG Anunoby","Terence Davis","Terence Davis","Pascal Siakam","Jeff Green","Serge Ibaka","Joe Ingles","Donovan Mitchell","Norman Powell","Joe Ingles","Pascal Siakam","Kyle Lowry","Donovan Mitchell","OG Anunoby","Joe Ingles","Fred VanVleet","Joe Ingles","Tony Bradley","Tony Bradley","Serge Ibaka","Serge Ibaka","Rudy Gobert","Rudy Gobert","Pascal Siakam","Pascal Siakam","Donovan Mitchell","Joe Ingles","Pascal Siakam","OG Anunoby","Donovan Mitchell","Royce O'Neale","Serge Ibaka","Jeff Green","Pascal Siakam","Rudy Gobert","Kyle Lowry","Kyle Lowry","Donovan Mitchell","Marc Gasol","Serge Ibaka","Joe Ingles","Rudy Gobert","Joe Ingles","Joe Ingles","Rondae Hollis-Jefferson","Donovan Mitchell","Kyle Lowry","Royce O'Neale","Tony Bradley","Kyle Lowry","Bojan Bogdanović","Kyle Lowry","Pascal Siakam","Pascal Siakam","Emmanuel Mudiay","Fred VanVleet","Royce O'Neale","Terence Davis","Serge Ibaka","Serge Ibaka","Serge Ibaka","Serge Ibaka","Rudy Gobert","Rudy Gobert","Kyle Lowry","Kyle Lowry","Donovan Mitchell","Mike Conley","OG Anunoby","Pascal Siakam","Donovan Mitchell","Royce O'Neale","Kyle Lowry","Kyle Lowry","Donovan Mitchell","Donovan Mitchell","Fred VanVleet","Royce O'Neale","Donovan Mitchell","OG Anunoby","Rudy Gobert","Fred VanVleet","Rudy Gobert","Kyle Lowry","Kyle Lowry","Royce O'Neale","Rudy Gobert","Pascal Siakam","Pascal Siakam","Rudy Gobert","Rudy Gobert","Serge Ibaka","Donovan Mitchell","Kyle Lowry","Chris Boucher","Chris Boucher","Joe Ingles","Mike Conley","Bojan Bogdanović","Bojan Bogdanović","Norman Powell","Rudy Gobert","Rudy Gobert","Serge Ibaka","Kyle Lowry","Bojan Bogdanović","Bojan Bogdanović","Pascal Siakam","Donovan Mitchell","Kyle Lowry","Donovan Mitchell","Donovan Mitchell","Kyle Lowry","Kyle Lowry","Donovan Mitchell","OG Anunoby","Jordan Clarkson","Rudy Gobert","Ed Davis","Ed Davis","Chris Boucher","Pascal Siakam","Pascal Siakam","Rudy Gobert","Rudy Gobert","Kyle Lowry","Serge Ibaka","Serge Ibaka","Emmanuel Mudiay","Fred VanVleet","Terence Davis","Bojan Bogdanović","Bojan Bogdanović","Kyle Lowry","Terence Davis","Joe Ingles","OG Anunoby","Terence Davis","Emmanuel Mudiay","Emmanuel Mudiay","Fred VanVleet","Jordan Clarkson","Rudy Gobert","Rudy Gobert","Rondae Hollis-Jefferson","Joe Ingles","Bojan Bogdanović","Bojan Bogdanović","Serge Ibaka","Serge Ibaka","Serge Ibaka","Donovan Mitchell","Donovan Mitchell","Pascal Siakam","Joe Ingles","Georges Niang","Pascal Siakam","OG Anunoby","Royce O'Neale","Rudy Gobert","Pascal Siakam","Donovan Mitchell","Patrick McCaw","Kyle Lowry","Kyle Lowry","Kyle Lowry","Joe Ingles","Joe Ingles","OG Anunoby","Kyle Lowry","Royce O'Neale"],["2pt","3pt","Rebound","3pt","Rebound","FT","FT","3pt","3pt","Rebound","Turnover","3pt","Rebound","3pt","Rebound","2pt","Rebound","3pt","Rebound","3pt","Rebound","3pt","Rebound","2pt","3pt","2pt","Rebound","3pt","Rebound","3pt","2pt","Rebound","3pt","3pt","Rebound","FT","FT","2pt","3pt","Rebound","2pt","Rebound","Turnover","2pt","3pt","Rebound","3pt","Rebound","3pt","Rebound","3pt","2pt","Rebound","2pt","Rebound","FT","FT","Rebound","3pt","Rebound","2pt","Rebound","Turnover","2pt","2pt","Rebound","2pt","2pt","FT","FT","2pt","Rebound","3pt","FT","FT","Rebound","Turnover","Turnover","FT","FT","3pt","Rebound","FT","FT","Rebound","2pt","Turnover","3pt","Rebound","2pt","3pt","Rebound","2pt","2pt","3pt","Rebound","2pt","3pt","Rebound","FT","FT","Turnover","2pt","FT","FT","3pt","Rebound","3pt","3pt","Rebound","FT","FT","2pt","Rebound","Turnover","3pt","3pt","Rebound","2pt","FT","FT","3pt","FT","FT","Rebound","FT","FT","FT","FT","2pt","Rebound","2pt","Rebound","3pt","2pt","2pt","3pt","2pt","Rebound","FT","FT","Rebound","3pt","Rebound","2pt","Rebound","3pt","3pt","Rebound","3pt","FT","FT","Rebound","2pt","FT","FT","FT","FT","2pt","Rebound","FT","FT","2pt","Rebound","Turnover","3pt","Rebound","2pt","2pt","Rebound","2pt","Rebound","2pt","Rebound","3pt","3pt","FT","FT","2pt","Rebound","2pt","Rebound","3pt","Rebound","FT","FT","2pt","Rebound","2pt","Rebound","2pt","Turnover","2pt","Rebound","2pt","2pt","2pt","Rebound","Turnover","Turnover","2pt","3pt","2pt","3pt","Rebound","2pt","Rebound","2pt","FT","FT","Rebound","2pt","Rebound","3pt","Rebound","2pt","Rebound","3pt","2pt","Turnover","Turnover","2pt","FT","FT","2pt","Rebound","2pt","3pt","Rebound","FT","FT","2pt","3pt","3pt","Rebound","2pt","Turnover","2pt","Rebound","FT","FT","2pt","2pt","3pt","Rebound","FT","FT","Rebound","3pt","FT","FT","FT","FT","Rebound","2pt","Rebound","2pt","Rebound","Turnover","FT","FT","Rebound","2pt","3pt","Rebound","2pt","2pt","Turnover","3pt","2pt","FT","FT","Rebound","2pt","FT","FT","FT","FT","2pt","3pt","3pt","Rebound","2pt","3pt","Rebound","FT","FT","3pt","Rebound","2pt","Rebound","2pt","FT","FT","2pt","3pt","3pt","FT","FT","FT","FT","2pt","2pt","3pt","Rebound","FT","FT","Rebound","FT","FT","FT","FT","Rebound","FT","FT","2pt","Rebound","3pt","Rebound","2pt","Rebound","Turnover","3pt","Rebound","2pt","FT","FT","2pt","Rebound","FT","FT","2pt","Rebound","FT","FT","Rebound","FT","FT","FT","FT","2pt","Rebound","2pt","Rebound","2pt","Rebound","2pt","Turnover","3pt","2pt","Rebound","FT","FT","FT","FT","Rebound","3pt","Rebound"],[2,0,0,0,0,1,1,3,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,2,3,0,0,0,0,3,0,0,3,0,0,0,1,2,0,0,0,0,0,2,0,0,0,0,0,0,3,0,0,0,0,1,0,0,0,0,0,0,0,2,0,0,2,2,1,1,0,0,3,1,0,0,0,0,0,1,0,0,1,0,0,2,0,0,0,2,0,0,2,2,0,0,2,0,0,0,1,0,2,1,1,0,0,3,0,0,1,1,0,0,0,3,0,0,2,1,1,3,1,0,0,0,1,1,1,0,0,0,0,3,2,2,3,0,0,0,0,0,0,0,0,0,3,0,0,3,0,0,0,2,1,1,1,1,0,0,0,1,0,0,0,0,0,2,0,0,0,0,0,0,3,3,0,1,0,0,0,0,0,0,1,1,0,0,0,0,2,0,0,0,2,2,0,0,0,0,2,3,2,0,0,0,0,2,1,0,0,0,0,0,0,0,0,3,2,0,0,2,1,1,0,0,2,0,0,1,1,2,3,0,0,2,0,0,0,1,1,2,2,0,0,1,0,0,3,0,1,1,0,0,0,0,0,0,0,1,0,0,2,0,0,2,2,0,3,2,1,0,0,2,1,1,1,1,2,3,0,0,2,0,0,1,1,0,0,0,0,2,0,1,2,3,3,1,1,1,1,2,2,0,0,1,0,0,1,1,0,0,0,1,1,0,0,0,0,0,0,0,0,0,2,1,1,0,0,0,1,0,0,1,0,0,1,1,1,1,0,0,0,0,0,0,2,0,3,0,0,1,1,0,0,0,0,0]],"container":"<table class=\"display\">\n  <thead>\n    <tr>\n      <th> <\/th>\n      <th>Team<\/th>\n      <th>Player<\/th>\n      <th>Event<\/th>\n      <th>Points<\/th>\n    <\/tr>\n  <\/thead>\n<\/table>","options":{"columnDefs":[{"className":"dt-right","targets":4},{"orderable":false,"targets":0}],"order":[],"autoWidth":false,"orderClasses":false}},"evals":[],"jsHooks":[]}</script>
</div>
<div id="monte-carlo-simulation" class="section level1">
<h1>Monte Carlo Simulation</h1>
<div class="sourceCode" id="cb8"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb8-1" data-line-number="1"><span class="co"># Monte Carlo Simulation</span></a>
<a class="sourceLine" id="cb8-2" data-line-number="2">home_team &lt;-<span class="st"> &quot;LAL&quot;</span></a>
<a class="sourceLine" id="cb8-3" data-line-number="3">away_team &lt;-<span class="st"> &quot;UTA&quot;</span></a>
<a class="sourceLine" id="cb8-4" data-line-number="4">nPossessions &lt;-<span class="st"> </span><span class="dv">100</span></a>
<a class="sourceLine" id="cb8-5" data-line-number="5">nGames &lt;-<span class="st"> </span><span class="dv">20</span></a>
<a class="sourceLine" id="cb8-6" data-line-number="6"></a>
<a class="sourceLine" id="cb8-7" data-line-number="7">home_team_scores &lt;-<span class="st"> </span><span class="ot">NULL</span></a>
<a class="sourceLine" id="cb8-8" data-line-number="8">away_team_scores &lt;-<span class="st"> </span><span class="ot">NULL</span></a>
<a class="sourceLine" id="cb8-9" data-line-number="9">playByPlay_overall &lt;-<span class="st"> </span><span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Player&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Event&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="kw">double</span>(), <span class="dt">stringsAsFactors =</span> F, <span class="dt">game =</span> <span class="kw">double</span>()) </a>
<a class="sourceLine" id="cb8-10" data-line-number="10"></a>
<a class="sourceLine" id="cb8-11" data-line-number="11"><span class="cf">for</span>(j <span class="cf">in</span> <span class="dv">1</span><span class="op">:</span>nGames){</a>
<a class="sourceLine" id="cb8-12" data-line-number="12">  playByPlay &lt;-<span class="st"> </span><span class="kw">data.frame</span>(<span class="st">&quot;Team&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Player&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Event&quot;</span> =<span class="st"> </span><span class="kw">character</span>(), <span class="st">&quot;Points&quot;</span> =<span class="st"> </span><span class="kw">double</span>(), <span class="dt">stringsAsFactors =</span> F) </a>
<a class="sourceLine" id="cb8-13" data-line-number="13">  <span class="cf">for</span>(i <span class="cf">in</span> <span class="dv">1</span><span class="op">:</span>nPossessions){</a>
<a class="sourceLine" id="cb8-14" data-line-number="14">    <span class="kw">possession</span>(<span class="dt">team_A =</span> home_team, <span class="dt">team_B =</span> away_team, <span class="dt">printPlayByPlay =</span> F, <span class="dt">def_multiplier =</span> <span class="dv">1</span>)</a>
<a class="sourceLine" id="cb8-15" data-line-number="15">    <span class="kw">possession</span>(<span class="dt">team_A =</span> away_team, <span class="dt">team_B =</span> home_team, <span class="dt">printPlayByPlay =</span> F, <span class="dt">def_multiplier =</span> <span class="dv">1</span>)</a>
<a class="sourceLine" id="cb8-16" data-line-number="16">  }</a>
<a class="sourceLine" id="cb8-17" data-line-number="17">  playByPlay_overall &lt;-<span class="st"> </span><span class="kw">rbind</span>(playByPlay_overall, <span class="kw">cbind</span>(playByPlay, <span class="kw">data.frame</span>(<span class="dt">game =</span> j)))</a>
<a class="sourceLine" id="cb8-18" data-line-number="18">  <span class="co">#print(paste0(j, &quot;/&quot;, nGames))</span></a>
<a class="sourceLine" id="cb8-19" data-line-number="19">}</a>
<a class="sourceLine" id="cb8-20" data-line-number="20">playByPlay_overall2 &lt;-<span class="st"> </span><span class="kw">data.table</span>(playByPlay_overall)</a>
<a class="sourceLine" id="cb8-21" data-line-number="21">x &lt;-<span class="st"> </span>playByPlay_overall2[,.(<span class="kw">sum</span>(Points)), by =<span class="st"> </span><span class="kw">c</span>(<span class="st">&quot;Team&quot;</span>, <span class="st">&quot;game&quot;</span>)] <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">dcast.data.table</span>(game <span class="op">~</span><span class="st"> </span>Team)</a>
<a class="sourceLine" id="cb8-22" data-line-number="22"></a>
<a class="sourceLine" id="cb8-23" data-line-number="23"><span class="kw">print</span>(x)</a></code></pre></div>
<pre><code>##     game LAL UTA
##  1:    1 112 110
##  2:    2  92 100
##  3:    3 101  67
##  4:    4 109  96
##  5:    5  79  89
##  6:    6 120  72
##  7:    7 105  94
##  8:    8 103  93
##  9:    9  95  98
## 10:   10  89  84
## 11:   11  87  96
## 12:   12 103  78
## 13:   13  89  67
## 14:   14 112  99
## 15:   15 101  88
## 16:   16  82  73
## 17:   17  91  75
## 18:   18  99  96
## 19:   19 102  83
## 20:   20  94  84</code></pre>
<p>See which team won more games and by how much</p>
<div class="sourceCode" id="cb10"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb10-1" data-line-number="1">x<span class="op">$</span>spread &lt;-<span class="st"> </span>x<span class="op">$</span>LAL <span class="op">-</span><span class="st"> </span>x<span class="op">$</span>UTA</a>
<a class="sourceLine" id="cb10-2" data-line-number="2">x2 &lt;-<span class="st"> </span>x[,win <span class="op">:</span><span class="er">=</span><span class="st"> </span><span class="kw">ifelse</span>(spread <span class="op">&gt;</span><span class="st"> </span><span class="dv">0</span>, <span class="dv">1</span>, <span class="dv">0</span>)]</a></code></pre></div>
<p>Lakers won 16/20 games against the Jazz, and win by 11.15 points.</p>
<p>Average points per game accross 20 games data viz</p>
<div class="sourceCode" id="cb11"><pre class="sourceCode r"><code class="sourceCode r"><a class="sourceLine" id="cb11-1" data-line-number="1">scores &lt;-<span class="st"> </span>playByPlay_overall2[,.(<span class="kw">sum</span>(Points)), by =<span class="st"> </span><span class="kw">c</span>(<span class="st">&quot;Team&quot;</span>, <span class="st">&quot;Player&quot;</span>)]</a>
<a class="sourceLine" id="cb11-2" data-line-number="2">scores<span class="op">$</span>PPG &lt;-<span class="st"> </span>scores<span class="op">$</span>V1<span class="op">/</span>nGames</a>
<a class="sourceLine" id="cb11-3" data-line-number="3">p1 &lt;-<span class="st"> </span>scores[Team <span class="op">==</span><span class="st"> </span>home_team,] <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> Player, <span class="dt">y =</span> PPG)) <span class="op">+</span><span class="st"> </span><span class="kw">geom_col</span>() <span class="op">+</span><span class="st"> </span><span class="kw">coord_flip</span>() <span class="op">+</span><span class="st"> </span><span class="kw">theme_bw</span>() <span class="op">+</span><span class="st"> </span><span class="kw">theme</span>(<span class="dt">axis.title =</span> <span class="kw">element_blank</span>(), <span class="dt">panel.border =</span> <span class="kw">element_blank</span>())</a>
<a class="sourceLine" id="cb11-4" data-line-number="4">p2 &lt;-<span class="st"> </span>scores[Team <span class="op">==</span><span class="st"> </span>away_team,] <span class="op">%&gt;%</span><span class="st"> </span><span class="kw">ggplot</span>(<span class="kw">aes</span>(<span class="dt">x =</span> Player, <span class="dt">y =</span> PPG)) <span class="op">+</span><span class="st"> </span><span class="kw">geom_col</span>() <span class="op">+</span><span class="st"> </span><span class="kw">coord_flip</span>() <span class="op">+</span><span class="st"> </span><span class="kw">theme_bw</span>() <span class="op">+</span><span class="st"> </span><span class="kw">theme</span>(<span class="dt">axis.title =</span> <span class="kw">element_blank</span>(), <span class="dt">panel.border =</span> <span class="kw">element_blank</span>())</a>
<a class="sourceLine" id="cb11-5" data-line-number="5">p1 <span class="op">|</span><span class="st"> </span>p2</a></code></pre></div>
<p><img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAqAAAAHgCAYAAAB6jN80AAAEGWlDQ1BrQ0dDb2xvclNwYWNlR2VuZXJpY1JHQgAAOI2NVV1oHFUUPrtzZyMkzlNsNIV0qD8NJQ2TVjShtLp/3d02bpZJNtoi6GT27s6Yyc44M7v9oU9FUHwx6psUxL+3gCAo9Q/bPrQvlQol2tQgKD60+INQ6Ium65k7M5lpurHeZe58853vnnvuuWfvBei5qliWkRQBFpquLRcy4nOHj4g9K5CEh6AXBqFXUR0rXalMAjZPC3e1W99Dwntf2dXd/p+tt0YdFSBxH2Kz5qgLiI8B8KdVy3YBevqRHz/qWh72Yui3MUDEL3q44WPXw3M+fo1pZuQs4tOIBVVTaoiXEI/MxfhGDPsxsNZfoE1q66ro5aJim3XdoLFw72H+n23BaIXzbcOnz5mfPoTvYVz7KzUl5+FRxEuqkp9G/Ajia219thzg25abkRE/BpDc3pqvphHvRFys2weqvp+krbWKIX7nhDbzLOItiM8358pTwdirqpPFnMF2xLc1WvLyOwTAibpbmvHHcvttU57y5+XqNZrLe3lE/Pq8eUj2fXKfOe3pfOjzhJYtB/yll5SDFcSDiH+hRkH25+L+sdxKEAMZahrlSX8ukqMOWy/jXW2m6M9LDBc31B9LFuv6gVKg/0Szi3KAr1kGq1GMjU/aLbnq6/lRxc4XfJ98hTargX++DbMJBSiYMIe9Ck1YAxFkKEAG3xbYaKmDDgYyFK0UGYpfoWYXG+fAPPI6tJnNwb7ClP7IyF+D+bjOtCpkhz6CFrIa/I6sFtNl8auFXGMTP34sNwI/JhkgEtmDz14ySfaRcTIBInmKPE32kxyyE2Tv+thKbEVePDfW/byMM1Kmm0XdObS7oGD/MypMXFPXrCwOtoYjyyn7BV29/MZfsVzpLDdRtuIZnbpXzvlf+ev8MvYr/Gqk4H/kV/G3csdazLuyTMPsbFhzd1UabQbjFvDRmcWJxR3zcfHkVw9GfpbJmeev9F08WW8uDkaslwX6avlWGU6NRKz0g/SHtCy9J30o/ca9zX3Kfc19zn3BXQKRO8ud477hLnAfc1/G9mrzGlrfexZ5GLdn6ZZrrEohI2wVHhZywjbhUWEy8icMCGNCUdiBlq3r+xafL549HQ5jH+an+1y+LlYBifuxAvRN/lVVVOlwlCkdVm9NOL5BE4wkQ2SMlDZU97hX86EilU/lUmkQUztTE6mx1EEPh7OmdqBtAvv8HdWpbrJS6tJj3n0CWdM6busNzRV3S9KTYhqvNiqWmuroiKgYhshMjmhTh9ptWhsF7970j/SbMrsPE1suR5z7DMC+P/Hs+y7ijrQAlhyAgccjbhjPygfeBTjzhNqy28EdkUh8C+DU9+z2v/oyeH791OncxHOs5y2AtTc7nb/f73TWPkD/qwBnjX8BoJ98VQNcC+8AAEAASURBVHgB7J0JvE1V//+/UghFFIWE0qQBlR4NjykpU5ooTVTSpDSo+KWZBlOTJkoRFT2lNFEhpTRISUJoNBSVJhHy917Ps85/33PPOfecs/e97vD5vl7nnnP2Xnuttd/73LU/+7u+a61Sm7eYyURABERABERABERABESggAhsU0DlqBgREAEREAEREAEREAERcAQkQPVDEAEREAEREAEREAERKFACEqAFiluFiYAIiIAIiIAIiIAISIDqNyACIiACIiACIiACIlCgBCRACxS3ChMBERABERABERABEZAA1W9ABERABERABERABESgQAlIgBYobhUmAiIgAiIgAiIgAiIgAarfgAiIgAiIgAiIgAiIQIESkAAtUNwqTAREQAREQAREQAREQAJUvwEREAEREAEREAEREIECJSABWqC4VZgIiIAIiIAIiIAIiIAEqH4DIiACIiACIiACIiACBUpAArRAcaswERABERABERABERABCVD9BgolgY0bN9o///wTum6bNm2KJA/qE9Y4n82bN4fKhuOpSxT5RMFX1ynx5YzqOiXOXVuLGgH/e4jif86fexRtWzCvKNo4n18UbZ3Pq7Cz41yjZhfl7ySqNtpfjyh/dxKgnqreCxWBn376yf7444/Qdfr1119D50E9fv7550jyCdtQbdiwwVatWhW6wSOfP//8M/Q5cZ2iyEfXKfSlUAaFmACCgv/bv//+O7JaRvE/4ysTVRvn81u7dm1k54oAhd26det89qHfo2ZHOxiV/fXXX5Gxo06wI8+oLEp2EqBRXRXlIwIiIAIiIAIiIAIikBYBCdC0MCmRCIiACIiACIiACIhAVAQkQKMiqXxEQAREQAREQAREQATSIrBtWqmUSAS2AoHOnTunLPW1115LuV87RUAEREAEREAECicBCdDCeV1UKxEQAREQgWJGoFOnTinPSA/VKfFoZzEjoC74YnZBdToiIAIiIAIiIAIiUNgJSIAW9iuk+omACIiACIiACIhAMSMgAVrMLqhORwREQAREQAREQAQKOwEJ0MJ+hVQ/ERABERABERABEShmBCRAi9kF1emIgAiIgAiIgAiIQGEnIAFa2K+Q6icCIiACIiACIiACxYyABGgxu6A6HREQAREQAREQAREo7AQkQAv7FVL9REAEREAEREAERKCYEdBE9PlwQTdu3GivvPJKwpz32GMPO/jggxPuy3bjzJkz7aeffnKHlylTxnbZZRdr1KiRbbNN9s8Xn3/+uZFX/fr1k1Zr9erV9sUXX9jRRx+dNI12iIAIiIAIiIAIiEA8AQnQeCIRfN+0aZMtWrTI5fTDDz8YYq5ly5bu+/bbbx9BCTmzGDdunG233XZWq1Yt++uvv2zx4sX2zz//2P3332+VKlXKmTjNb/Pnz7cKFSqkFKDfffedPfnkkxKgaTJVMhEQAREQAREQgf8SkADNh19C2bJl7eqrr3Y5v/fee7Zq1arYdzauXbvWtt12W8ODWK1aNfeZ7b/88ov9/fffVr16db7a77//buXKlXPiku8bNmywdevW2Q477MDXHNauXTtr3bq124YAvvXWW23o0KF28803x7atWLHCCdMaNWrEyvz111+dSEUoY77s9u3bW6lSpdw2/qxfv96WLVvm9iNMExn1JR/yL1++fKIk2iYCIiACIiACIiACJgG6FX4EjzzyiH3zzTfOS3rsscfaGWecYQMHDjTE4Jo1a6xevXp2xx13GOnwYJ5//vmulng6//jjD7vkkktS1rp06dJ28sknW+/evZ3gpKz+/fs7sUtXPWKSvHfccUfr1q2bHXjggU44Ih5btGhhl19+uT366KNu/9lnn22zZ892QpZ6LVy40C699FJD8Abt9ddftxEjRtiee+5pCxYssIsvvjgmiElH3Xl5GzRokO28887+a653PLh5GcI+L0OMp5MuVT7ksXnz5kjy4QEiKOxTlZtoH/XAeFgJmw95UZ8wxnX6888/Q+dTUq4TD488VBYF8w/P8XWlt+X444+P3xzZ97feesu1hWRIGFCdOnVs3333DZ3/hx9+6B6Oa9asaay53qxZM8uPHqnQFVUGIlBCCEiAbqULzQ33pZdecsIGbyXd53gsiR9FOCL62rZtazfddFNMgE6e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ARHISYDYTha6YNARvTeE6dBm5WUMhBw8eHDKOHRiMWnzeJhlBDsj8uN7kiiHGFLiz5kbmbhDRCfziGI9e/Z0+whBQsgmWlYYAUp68mcwFbOMHHPMMe54/qRT11hifRABEciIgARoRrgKNjEN7wknnFCwhao0ERABEYgjwLRw8YbYY7YL4iaDvSOXXXZZjqQ33nhjju8IS4Rfsng+P6Aox0GBLy+88ELsG934LJpBDCO9L+RNFz9GNz1x6Yhj4t+CPTNMF+eNwUzE3dFFHx+zmFddfR56FwERyJyABGjmzHSECIiACIjA/wgExWcqKMSHEuv+7rvvGqPco7RUA0B8TGeq8uIHaORnXVPVQ/tEoCQRkAAtSVdb5yoCIiACW4kAg5UYtU63dqJ4zq1UrYTFFqW6JjwBbRSBIkBAArQIXCRVUQREQASKA4GOHTsWmdMoSnUtMlBVUREIENAo+AAMfRQBERABERABERABEch/AhKg+c9YJYiACIiACIiACIiACAQIqAs+AEMfCxcB1ngOO3dp4Toj1UYEREAEREAERAAC8oDqdyACIiACIiACIiACIlCgBCRACxS3ChMBERABERABERABEVAXvH4DhZZA586dI6/ba6+9FnmeylAEREAEREAERCAzAvKAZsZLqUVABERABERABERABEISkAc0JEAdLgIiIAIiIALpEGDp0nLlyqWTVGlEoNgTkAe02F9inaAIiIAIiIAIiIAIFC4CEqCF63qoNiIgAiIgAiIgAiJQ7AmoC77YX2KdoAiIgAiIQGEg0KlTp8JQDdWhkBIoaYNk5QEtpD9EVUsEREAEREAEREAEiisBCdDiemV1XiIgAiIgAiIgAiJQSAlIgBbSC6NqiYAIiIAIiIAIiEBxJaAY0Aiv7N9//23BGI5SpUrZLrvsYvvss4/ttNNOKUv6+eefbd68efbvf/87Zbp0doapRzr5k2bz5s02adIk69Chg3GeMhEQAREQAREQARFIl4A8oOmSSiPdX3/9ZYMGDbLvvvvOli9f7t6ffPJJO+ecc+zrr79OmQMC9K233kqZJt2dYeqRbhn//POPO1eEqEwEREAEREAEREAEMiEgD2gmtNJMe/7551vZsmVjqfv162eTJ0+2nj17um0IRARqpUqVbOedd3bb6tSpY717944dw4dffvnF8GZWr17dbd+4caOtW7fOtt9+eydu8a5WqFAhxzHBL3nVg7SI5SpVqsTyyasM6rNy5UqrUaNGsCj3ecOGDe68atasadtuq59WLkDaIAIiIAIiIAIi4AhIJeTzD2HTpk22du1a23XXXV1JL774oo0ZM8bq1atn8+fPd13uffr0cZ/vvfdeGzlypK1evdoGDhxov/76q61Zs8alveOOO2zBggU2dOhQw/u4ww472JIlS+yWW26xQw89NM+ziK/HO++8Yw8//LBVrlzZFi1aZBdeeKGdeOKJKcuYMWOGUY+6desa+QVt+PDh9vbbb7v8Vq1a5byjnKM36v7FF1/4r+68EdLJLL88q1yLTA1RTn2yOTZYFgKdcAXeszXqgvEgEjYf8gp7TnChHmHz8b/PbLlwXFG4TmXKlNHDWZiLrGNFQASKDQEJ0Hy4lDfccIMTGtwQ8XTuu+++1rp1aydiPvzwQ7vnnnucB3HFihXWpUsXu+qqq3LUAjFXq1YtJzbJA8/o7NmzrXz58vbll1/a6NGjnQgcO3asIWiTCdBk9cADe9ttt9ntt99ujRo1sh9++MHwlhLPiSUqo2HDhu4Y6r7ffvvZK6+8Yp9//rlLjzCdNm2aEW7AMnPjx4+3CRMm2LXXXuv28+f111+3+++/P/b96aefjony2MYC+ICoz9bCHOvLxIMchf3xxx9RZGPr168PnQ95RJFPFHw5mSjyya/rxAOfegdC/+SUgQiIQDEgIAGaDxfxhBNOsG222caJLrxDeDi9t++aa65xsZ6INISe9yAFq4Gw69u3r9vEzapZs2b2xhtvWMeOHZ1owwOJ7b777jZnzhz3OdGfZPXAG4knBlGJ0cVPXh999JFVrFgxYRnEsBJWgJjGjjjiCPfOH+rLd7/GccuWLe3ss892wtrfbPGwnnvuubFjPI/YhrgPeFHzw3w4QyZ5//bbby4UwodLZHJsMO3vv//ufgeeSXBfup8RRoRmEDax3XbbpXtYrnT8LhGNXO8wxnXiWobNh3PKa6BeXvUsCtdJA/byuoraLwIiUFIIaBBSPlzpQw45xP71r39Z//79nWjD24jRbXreeee5rvPDDz/cbrzxxoSlI3SC3auIDt/lveOOOyY8JtHGZPUgf7rxeXlDjKQqA7HDfn9M6dKlncjmeGJR4+tLnRHh3jieeFX/Yl+qlz8u6vdUZSbb50VDsv3pbicfXummT5YOJsn2pbs9yrpEcU5R5VHY2MRfD85TJgIiIAIisOU+Jgj5S4Bu6I8//thNz4S3iO7BSy65xJo2bRrzXnrh52uCB3HKlCkupo3YuunTp1uDBg387qzeg/VAgNauXdtmzpzp8lq6dKnxYrqoZEZ6vKazZs1ySehy92K0efPm9v777zvPHDuZiopuem6+MhEQAREQAREQARGIJ6Au+HgiEX+vWrWqXXzxxXbfffe5GEm60/GC0mVZZ8vId0aTf//99zlKJV6UuEriQxGnRx99tLGGMIOWsrVgPfC+9ujRw26++WY36Imu4ZtuusmNyGeEeyLDczNgwADntX3ooYdct70f6Y+gbdeunZ122mlGOZwbaWUiIALFgwAhOHPnzo2dDA+ju+22m3swzjSkhDh42j1my8jEvv32W1cHwjX23HNP18uU10NuVPMr00Pk27tM6qy0IiACyQmU2hKDqIkck/PJlz14NenCDjZoxF8+/vjjOQbqIAyJqwwT65fXCWQTe0esXaJQALrhCTNghH5YY2AU86dGbcGFAtLNG681N6Bq1aqle0jCdHAjXjLM9SS04aeffnIPC2Hz4ZzCXiuuE4PjwuaDUCCuNYwVx+sUhkeUxz7//PP2zDPPGL0d2J9//mkffPCBE6HDhg3LaDEKpqWjl+eYY45xeaXzh4GZ9NjwAM//0Lvvvut6ZAYPHuzek+WBaPaziyRLk9d2BoKeccYZ7vzzSptsP46EH3/80bp3754sibaLgOs95H8LfeDHVITFwmBn7teppmzMpIwo2mpfnjygnkQBvnPDDtrChQvdqPH69esHN4e+qefILMmXbAZ+JBKfZI8gCiOKklRRm0VABAoBgT322MNN1+arwgPySSedZIsXL3azcjC7hn8Q4UGJB9LgTY85h4kX98YDEOIumIZp55gfORgrO3HiRPv0009t1KhRsYf2M8880y6//HIXqtS+fXufZa55jf0O6rNs2TLndcV76w3/CzOV4MX1AxSpk68b+/nOTTybh3Vfjt5FQARyE5AAzc1kq2w59thj3dP9VilchYqACIhAhgQQnBhhN5999pkRmsPcwhiruhEnznzGeEwuvfRSJ07x3nsByOwZQ4YMMaaTw+hiv/rqq3N5GqdOnWqdO3eOiU/S0vV+6623xsRrsnmNScu8yvSm4GFHBN95550uRh0BzRR4CEymNaNbn/oS6sR0c4hhhCkhSwhRwoqYB9mHHOCRRZhibKMNT2Y+Xj7Zfm0XAQjQO+oH8Eb5myHP4ENdGNp484PzPuOpzSsUJll5EqDJyBTgdgb/pBoAVIBVUVEiIAIikJAAC0lcd911bh/dhIjJCy64wAm7b775JuExbHzggQfsqKOOcrHwhEngNcWYg5gbI9PCMb0bq8W1adMmx40S4cf+K664wh3DH7rV/Q2QeVXxzKaa15h6svgHMffMT0w8PnVC2NLrxDR53FQ5t3nz5rlyGJTJVHmIazy5L7zwgtHdH7QnnngitnwyXlyEqkwEwhDg/yM/jNA4XlFZsJ48UEqARkVW+YiACIiACOQiwKCjk08+2XkLH3vsMbdwxamnnporXfwGPIqIPIzu9QMOOMB9xiNz/PHHu250HsCZ+QPPY9BIg7jDG+mNRS2IPWZWEUKIEMHcBBPNa0y4E8KTF3bkkUc68Yl3CQ8t5gdM0sU+fcuMI8SZMkgqGC7gEsb9YWEN76Wint6zG5fMfUXg5tfcxonK07aiSYAVE6OOAWVgMaExwVCXMHQUAxqGno4VAREQARHImADexsMOO8wdh4ikW5sZMJi1A0NkefPd83wnLhxPpzffhc13BOhFF13kZvogBhPhF28HHnigm8oOjynmV47DY8orOK8xgzcwYjh9fYLeGbZTht/WpEkTO/jgg90x/OFGjdjNa6EM0qYSnOwPWlTdn8E89bn4EeB3EnxFdYY+zyjyizIvTdQYxRVRHiIgAiJQggjgHWQQ0NChQ11XPJ5NPC2+a9zPFwwShOObb77pvIXEY37yyScxUnhV6UIndhQxmshYVY2BSCw77LsRiRfFE4q4zWteY8ID/LLBrCiHhxNjJD7eWbyvzLM8btw4I8wg3rxgDi62EZ9G30VABDInIAGaOTMdIQIiIAIlngDxmiw4cffdd1u9evWscePG1rVrVzvrrLNc7KQH1LNnT9f9zD4W4Yif7aNt27ZudbgWLVr4Q3K877XXXsZUT3SZEz9KevJhSeIbbrjBpWVeY8SwXwLYz2vMThbRYHDR6aef7qZyYoliDAGKp/aUU06xbt26OY9moqmh8KoedNBBxnF0P8pEQASiIaB5QKPhqFwiJqB5QBMDpStT84AmZqN5QBNzKcitjCyn+9p7DYNls484NN/97fcR+zl79mzr27ev35T0nW515tNNNn1cqqmS6FonjCDeiLmjvsF5mePT8B3va7ZzM1JvzQOaiKq2BQkwT3XUMaCaBzRIWJ9FIE0CjEJNNudomlk4j0XYCc7TLUvpRKCkE/DzgCbiEL8PUcYk8Uwqz9RI6RjeyGTik+NT7UskPjkm3cEZ2YpPypCJgAjkJqBpmHIz0RYREAEREIF8JoCYZO5NutTpwpeJgAiULAISoCXreutsRUAERKDQEOjYsWOhqYsqIgIiULAEJEALlrdKy4AAq5/EWzZrucfnoe8iIAIiIAIiIAJbl4BGwW9d/ipdBERABERABERABEocAQnQEnfJdcIiIAIiIAIiIAIisHUJSIBuXf4qXQREQAREQAREQARKHAEJ0BJ3yXXCIiACIiACIiACIrB1CUiAbl3+Kl0EREAEREAEREAEShwBCdASd8l1wiIgAiIgAiIgAiKwdQloGqaty1+li4AIiIAIlBACEydOzHo5z3hErEsf1SpvUS1j6+v4xx9/2HbbbZfn8qY+far3f/75x1iauVKlSla+fPlUSdPeFyU7lob966+/rHr16mmXr4T/JSAPqH4JIiACIiACIiACIiACBUpAHtACxM2T3Jdffmlz5swx1kU+9NBD03pq+uWXX2zu3LnWrFmzyGq7du1a++yzz2zRokXWoEEDO+igg2zbbbP/OXz++edWpkwZq1+/fmR1VEYiIAIiIAIiIALFk4A8oAV0XTdt2mTXX3+9DRkyxNavX28LFiywCy64wJ5//vk8a4AAnTFjRp7p0k2wcuVK6969uz333HOGKH766aeta9euhijN1qZNm2bvvfdetofrOBEQAREQAREQgRJEIHuXVwmCFMWpPvjggy6bhx56yLbZ5r+6/7TTTrMLL7zQatWqZYcddpgTgKVLl47Fzfz5558ujqZ27drWu3dvdzwiEU/lxo0bXVzM7rvvHvNcEsdDnAzxMliymJQrr7zSTjzxRKN8bzfffLM9/PDDdsUVV/hN9t1337kYowoVKsS28WHDhg22fPlyq1mzZqzsYAJELXXByxvGqxrMU59FQAREoKgT6NSpU6E6BS1tXKguR4mrjARoAV3yt956y26//faY+KRYBFy7du2cdxMBijitUaNGTBgOGDDAWrRo4bYNHjzYRo0aZSNHjrQVK1bYV199ZRUrVnTBz4888oghErt162YHHnigE6CIUI69/PLLc5whonLNmjXWpUuXHNuvu+66WN3eeecdJ0YrV67suugRyQhWbPjw4fb2228b+1atWmWDBg2yevXqxfJCfN56660ur379+sW264MIiIAIiIAIiIAIeAISoJ5EPr7ThU63N97KeCNm8qmnnorfnPL7999/744pVaqU9ejRwz744AMnNjmoTp06dssttziRevrpp1uvXr1iwpL9dP1TD44NWtmyZd1XRvPddtttTiw3atTIidnzzz/fOnToYO+++67R1f7kk0+6kZzjx4+3CRMm2LXXXuuO3bx5sw0cONB5cK+55poc5SKceXkbMWKE7bLLLv5rrnfySmRwzMTIJ9Nj4vP3dYkinzBhDtTL12X16tW5rmF8vfP6Tl542cMYeTDiNYp8ouDLuUSRT35dJ3oott9++zDIdawIiIAIFAsCEqAFcBmZOoKuaMSdF3q+2HXr1mV8Q2ratGlMfOBFDd4sjzjiCJf1brvt5sQK+Qenrthpp51c97gvP/79m2++cYOJGjZs6HbRjY9g/eijj4yBRuRfrlw5t69ly5Z29tln21VXXeW+I0gRIo8++mgO8clOBjmdccYZLh1/dt55Z+e1jW2I+5BM0MSHA8QdlusrzMPe8InZJeQh07LjK0M+/A4Is8jWiCX25xQ2H84p/veYab24TpxT2Hz8OWVafjB9UbhOCkkJXjF9FgERKMkEJEAL4Opzc957771t9uzZ1qpVK1ciXeSIO7bts88+sVogMLxxU05kO+64Y2xzvCeTuEtv8fvYTj3owscrixj19uabbxphApdddpkbmERXuhc43NipFx7LJUuW+EPs77//di8f00oYAV5TQg0IJwjebJs0aWK80rWgqA4eEzy/4PZkn4lXzfSY+LxgwStsPngLEcPMj5etwZzfBWI4bD5c17DnxHXi9x02H12nbH8ROk4EREAEiiYBjYIvoOtGVzmxmnSB43kivrNPnz724YcfxjyDdM8R24n99NNPLv4y6uohXk8++WS74YYbYp5QymQA0rHHHus8kwx6mjlzpit66dKlxguR3Lx5c3v//fedeGUnAez77bdfzNtJLGjHjh2dyBo7dmzUVVd+IiACIiACIiACxYSAPKAFdCGZ85N4zKFDh7pBQHgUEXoM5sHzyOjI448/3o1CP/PMM52IO+CAA/Kldpdccondc889dt555znPHt60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kfDjjz+6+0JhFKBaCz7734jWgs+enTyg2bNLemQU4pPM010jOVlFEh2PMAwjPimLWNdEebMvbN7kIRMBERABERABESjeBBQDWryvr85OBERABERABERABAodAXlAC90lUYVEQAREQASKIwFi+Qva8goXKej6qDwR8ATkAfUk9C4CIiACIiACIiACIlAgBCRACwSzChEBERABERABERABEfAEJEA9Cb2LgAiIgAiIgAiIgAgUCAEJ0ALBrEJEQAREQAREQAREQAQ8AQlQT0LvIiACIiACIiACIiACBUJAo+ALBLMKyYbA+PHjQ88rGsVqIVFN0syExWHXBo9qgnPyWb9+fdL5XNO9XiwYUL58+dD5FLfrlC4/pRMBERCBkkpAHtCSeuV13iIgAiIgAiIgAiKwlQhIgG4l8CpWBERABERABERABEoqAXXBl9QrXwTOu3PnzkWgltlVUZNDZ8dNR4mACIiACBQPAvKAFo/rqLMQAREQAREQAREQgSJDQAK0yFxBBRhxAABAAElEQVQqVVQEREAEsiOwaNEimzNnTnYH/++oZcuW2Ycffhgqj/iD165da++//76NGTPGPv74Y9u4cWMsCb0Ef/31V+x7Oh9ef/11+/PPP9NJqjQiIAJbmYAE6Fa+ACpeBERABPKbwLvvvmuvvvpqqGLmz59vEydODJVH8OCVK1da9+7d7bnnnrN//vnHnn76aevatashSrHhw4cbM0dkYg8++KD98ssvmRyitCIgAluJgAToVgKvYkVABERgaxJAqDGNlje8j3gPmXZszZo1frMxRdbq1atj3/2HTZs22ffff2/ffvttzHNJHn/88Yex7+uvv07pjbzyyivtxBNPtDvvvNPOOeccu+uuu6xBgwb28MMP+yJyvFOPxYsXxwQqOxGrTCm2fPnyWB38QUwzFhSwpAnW1afbsGGDffPNNzmO9/ny/tVXX+XY54/TuwiIQDgCGoQUjp+OFgEREIEiRQAxOXDgwJjQrFevnt1xxx2Gh/Oee+5x4nPbbbd1HknEId3uzPXKa+edd3bniijr37+/VatWzX766Sc3p+wjjzziBN7QoUOdR3OHHXawJUuW2C233GKHHnpoDkbfffedK6dLly45tl933XW2zTY5/SKIWspCLFeqVMm++OIL9/3II480ykQ8EmJw7LHHxvIibe/eva1Hjx7WsmVL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iU9AEN8ZVnyST36IT/JNNMce22UiIAIiIAIiIAIiECSgUfBBGlvpMxO8E6/JvJoyERABERABERABESjuBCRAC8EVZqokmQiIgAiIgAiIgAiUFAISoCXlSus8RUAEREAEtioBFhApV65cJHWIck3uSCqkTEQgQwIaBZ8hMCUXAREQAREQAREQAREIR0ACNBw/HS0CIiACIiACIiACIpAhAXXBZwhMyUVABERABEQgGwLMdCJLj0BwoZb0jlCqokZAHtCidsVUXxEQAREQAREQAREo4gTkAS3iF7A4V3/8+PFuYvsw5xhFoH5hmuA8DAsdKwIiIAIiIAKFhYA8oIXlSqgeIiACIiACIiACIlBCCEiAlpALrdMUAREQAREQAREQgcJCQF3wheVKqB65CHTu3DnXtuKwQcH1xeEq6hxEQAREQATCEJAHNAw9HSsCIiACIiACIiACIpAxAQnQjJHpABEQAREQAREQAREQgTAEJEDD0NOxIiACIiACIiACIiACGROQAM0YmQ4QAREQAREQAREQAREIQ0ACNAw9HSsCIiACIiACIiACIpAxAQnQjJHpABEQAREQAREQAREQgTAEJEDD0NOxIiACIiACeRJYtGiRzZkzJ8906SaYPHmy/fLLL+kmz5WOqdD++uuvXNu1QQREoOAISIAWHGuVJAIiIAIlksC7775rr776amTn/sADD9gPP/yQdX7Dhw+33377LevjdaAIiEB4AqEE6FtvvWUrV67MUYv33nvP3n777RzbMv2yfv36TA9x6Xkipk4FYT/++KPxFP3MM8/Yt99+m1aRc+fOtSVLluRKu3nzZnvxxReN9zC2evXqGPtkT/iwpSz/4qbwySefhC47TL11rAiIQMkjQHudSESuWbPGvvrqK9u4cWOeUEjzxx9/2KZNm+zrr7+2P//8M9cx3333na1du9Z5PBPdW2h3ly1blrAuy5cvd+17OnXJVbA2iIAIpCQQaiWkJ554ws477zzbddddXSHPPvusjR071u65556UhabaybENGza0Zs2apUqWcB8N2owZM7I6NmGGSTa+8cYbdt9999kRRxxhVapUsX79+lmjRo3sqquuSnLEfzdzXI0aNWzPPffMke6ff/6xQYMGWfv27a1UqVI59mXyhYb2ySeftKOPPtp4wqdO22+/fY4saKwpq2PHjq6sDRs22OjRo129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udJ8+eWXNnv27FzbgxuYgPivv/4Kbkr5efHixfbiiy/ajz/+mCPdP//8Y5MmTbLPP/88x/ZkX7799lt76aWXbMyYMfbuu+8ax8tEQAREQAREQAREIEoCEqBZ0hw1apR99dVXKY+eM2eOPfPMM7nSPPDAA7Zy5cpc24Mbhg8fbr/99ltwU8rP77//vg0aNMiJ0GDCTz/91O666y6bNm1acHPCz5R53XXX2bJly2zz5s02evRo6927t/39998J02ujCIiACIiACIiACGRDQAI0G2oJjsFbuWTJElu9enVsb7t27Wzq1Km2cePG2Da8pgsWLLCWLVu6bZs2bbLvv//e8DwG08UO2PIBMYgozMvjuv/++7vygsfihd1zzz2Dm4wyKY93bxMnTjTEKsK6Z8+edvbZZxtCmbKnTJnik9n69ett6dKl9ueff8a2+Q9r1qxxojzZefh0ehcBESg5BNLpLYLGrFmz7NFHH3W9RrRPPADPnDkzISjamrffftsef/xxmzx5svFdJgIiULQIaC34CK4XXd90WderV8/mz59v//73v61Pnz6GINxpp53sww8/tKZNm7qSaCwRn9tvv70Ta/3797dq1arZTz/95MTdI488YjvuuGOsVr///rtdddVVTpz+8ccfTkwOHDjQSpUqFUvjP1D+vHnznBBGdCIE8cIeddRRsa70uXPnWr9+/ax+/fpO0F522WX2r3/9ywnXzp07W9myZX12ts0229itt95qFSpUcNsIG7j55pvdeS5cuNAuvfRSQ2RjDz30kL3yyitWq1YtW7VqlQ0ZMsRq167t9vGH8w6ua0y5lStXju2P/1BYuv5/+eUX27Bhg+PH5zBGPrzgmq15LnjHw+ZDXmEfFsiDh6+w+cAlCr7UJ4p88us68b9UpkyZbC9/kTyOh9qLLrrIqlevnrT+hBDRtpx55pmOz9VXX22NGze2gw8+ONcxtGnXX3+9NW/e3LU3tK8PPvig3XnnnbbPPvvkSq8NIiAChZOABGjI64KHkAbwnnvusRo1atiKFSusS5cuTjQiENq3b+88iF6AIsIQnRgN6WmnnebS8P388893saEtWrTgqzM8qIhFBC0eS7rIEZkHHnigT5LjvVWrVk5MIkA/+ugja9iwoW233XZO3JKQsmm8EZ1ffPGF/ec//7HDDz/ceWWvuOKKWF4I1bVr17rvCMV9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+1h+X1iih/ZEvSjZ1Jd8eIBHTOZlyXqR6PLM754bepUITejevXte1cyxn/YnmUUxc4TPO6o2zueHVzrKLnjuBzzYJHrA8GVm8h4lO9p1fj/x03tlUp9gWn6n/J6jitlkakj+R6JyYkXJTh7Q4JXXZxEQAREQgUgIJBKePuNkvSh+f7rvmeST7Aasnpt0aSudCERLIPslWaKth3ITAREQAREQAREQAREoIQQkQEvIhdZpioAIiIAIiIAIiEBhISABWliuhOohAiIgAiIgAiIgAiWEgARoCbnQOk0REAEREAEREAERKCwEJEALy5VQPURABERABERABESghBDQKPgScqGL4mmOHz8+rTlSU51blFNGpCpH+0RABERABERABNInIA9o+qyUUgREQAREQAREQAREIAICEqARQFQWIiACIiACIiACIiAC6ROQAE2flVIWMIHOnTsXcIkqTgREQAREQAREoCAISIAWBGWVIQIiIAIiIAIiIAIiECOgQUgxFPogAiIgAiIgAvlHYOLEiZGt8Z1/tVTOIlAwBOQBLRjOKkUEREAEREAEREAEROB/BCRA9VMQAREQAREQAREQAREoUALqgi9Q3CpMBERABESgpBLo1KlTZKc+bty4yPJSRiKwNQjIA7o1qKtMERABERABERABESjBBCRAS/DF16mLgAiIgAiIgAiIwNYgIAG6NairTBEQAREQAREQAREowQQUA5pPF3/69Om23377WfXq1SMvYdmyZbZ8+XI77LDDcuS9fv16mzx5cmxbhQoVrF69ela3bt3YtnQ/BMt47bXXrFmzZrb99tune7jSiYAIiIAIiIAIiEBSAvKAJkUTbseoUaPsq6++CpdJkqPnz59vzCcXb3/88YcNGjTIFi5c6F5Tp061yy+/3IYOHRqfNM/vwTKGDx9uv/32W57HKIEIiIAIiIAIiIAIpENAHtB0KEWQ5ueffzZeNWrUsPLly7sc165da9tuu62tXr3aqlWr5j5v3rzZeTfZHu895fh//vknz9pcccUVLi8Srly50i655BJr3LixNW/e3H799VfbYYcdbJtt/vvs8csvv9hOO+0UyzOdMtasWWMct/vuu7tyqBP5BvPh3EqXLm1ly5aN5a0P/6+9M4G3ctr//zeu5FJSaSCJSpShkCRTIRKuRBNJphDhUqr74xJSom4yVIZoUMrQTXKJ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qGA77bST4aFEJJ511llWpkyZSFksR3DWrFnTtm7dauXLl4+s++2332zNmjVWo0YNl6zO/qtWrXLeU4QmntRMjDIrVapkeHCx6Hpu27YtUo8ff/zRnSvH+umnn1xd99xzT6tSpYrblzpSL163b99uFStWdMvx4v7xxx/uvf6JgAiIgAiIgAiIQDwCEqDxyCRYXr16dRcqf+utt+ybb76xr776ynr16mXr16+3888/33ksEaeTJk2yLVu2OKGGKL344ouNfQYNGuTC+IhPxOzEiRPd0RBwCFuEHx5XtkMorlixwv79738bx6XcdOydd96xkSNHOpG4bNkyu+qqq6xt27bWu3dv6969ux111FH23HPP2fjx423q1KlOKPfs2dOdD3UYO3asqytezRNPPNEt5z1eWepG6J/6U9d58+bZn//8Z/fnxaqv6wsvvGDPP/+8/2i33nqrVa5cOfI5+k1xErKbNm2Krn7Mzwh2HiKCDyMxN0yw0HOBPQ85mRrl8PAQ7T1PtzzK4YEl23J4KEuVY7w6wpf6hFFOQV2nPfbYI/KwGu88tFwEREAESgMBCdAMr3L9+vVt8uTJ1q1bN7vppptcKR988IFVrVrVPvzwQycU58yZY1dffbXNmjXLreemdtddd9nw4cON/Vl+2223RWqAqHjkkUec0Bw9erRNnz7d/va3v7k8zzZt2sQVn9F5RRynY8eOznvJ8e655x5r3LixE8iXX365nX322da8eXN7//33nQCl3ojelStX2l577eWE5UEHHWTjxo1zQnOfffZxQrtDhw6Rc/3888+dwEZEzp4928g7RYjiDe7bt2/knPwbhh7xnlKWkaOajYDy5ebCa6rngfDkL9XtE50bZWRbzu+//551GdQxjHMKq4xcKgc20deJ+slEQAREQATMJEAz/BYQgt5tt91sv/32c2KKEDeC7qKLLrJXX33VeT7xjjZo0CAiQBFteEIRn9hxxx2XL6S+7777OvHJOjosIepSMYQfnkdvCFzsyy+/dOU3atTIfa5WrZqrL4LzhBNOsDvvvNMuu+wyo9PQ6aefbvPnz3flHH/88U5U4CUllxRPLjmseMzwVGGI0r333tu9xyN69NFHOw4saNasmfOGupX//de6dWvjL1XDm1xcDNGein3//ffuO4P4ztR4uMDDx/co23L4DgfTPzKpE9eJ30G25eD9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EAEREAEREAERKEACEqAFCFdFi4AIiIAIiIAIiIAI7EhAveB3ZJLREqafW7RokdvXjyvHVHDJxqDL6GBp7rR582ZXt5UrV7qp8Jo2bepmJkmzGG0uAiIgAiIgAiIgAqEQkAc0FIzmZssYN26cLVu2zD755BObMmWKtW/f3t59992QjpBZMQsWLLALL7zQmCN51113tXnz5tkll1zi5mXOrETtJQIiIAIiIAIiIALZEZAHNDt++fauW7eu3XzzzZFls2bNsoEDBxrCdM8993TLmcaKP+ZV9rOMMN0dM4T89ttvbsox5mH2n7dt2+ammFuzZo3tvffebjotpiZj2sG99torcqxYU+atXbvWbrvtNhs2bJhRN2/PPvusjRo1yu6//353TKbN49jMl8yczRhe0++++87N2xyc3o5tv/76azfXs5/ai31j1dMfT68iIAIiIALmHBPlypUTChEQgTwCEqAF+DVo0aKFE38LFy60Zs2a2a233hqZ03bx4sXuc/Pmze2xxx4z5msljM/8yT/99JMTiHweMmSIITgJ5RNC79+/vzVq1Mh5MVnHbB3YNddcY71797YGDRpEzuj999+3ww8/PJ/4ZOVf/vIXO+2009x2n332masjghOhOXHiRBs5cqRNnz7datasaRs3bnRCtVatWs7Le8cdd7iZQZYuXWrXXnuttWnTxpYsWRKznkcffXSkLqtWrTL+vHEOiRpixHBpMIR7OobYx3gQYLrATI1y/INDpmWwH9cpjHI4l3RZRNebMqhPtuVwPr/88ktWfJkPHYu+TkxRySw7MhEQAREo7QQkQAv4G3DwwQfb6tWrnccTj+ETTzzhjvj000/bK6+8YghQ7KuvvrIJEyZYmTJl7IorrjDEIx7P5cuX25gxY+yAAw6w8ePH29SpUw1h16pVK3v11Vft6quvthUrVjgREBSflPnpp59GBCqf8VxyHC/uvED8/PPPbdKkSVa5cmXn+ZwxY4bhJS1btqxNmzbNPbX37NnTxo4d6zyq7IcXFwHKVHVYvHq6lXn/XnzxRXvwwQf9Ryd0q1evHvlcWt/gZc7EtmzZksluO+yTrVijQMoIo5xMWUSfVBjlIBzDsOjrRIRht912C6NolSECIiACxZqABGgBXz5uZNxw8FSSi4mHEbFGnmjt2rUjR8dDivjE9t13XyOkjiHSEJ8YoXlyOjGE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G2azZs0insZ8G8f5QOckPJLcGHv37m1vvvmm0aFx+fLlTpQijho1auS8aV988YUTljVr1szX+al8+fLWtGlTo44cH8/poYcemu+IF110Ub7PiT4g7rhxI8pIBQjDKDOssnxHy7DKw+PG9UOYZ2uIdwQo1xRveBgWJjvSOahfWOzw8iIY8ciHYXg+YUducxgWJjsJ0DCuiMoQAREQARFIi0ClSpV26IREAQgXPHLe0vUELVq0yOrWresE32WXXWbNmzd3YrJz587Wrl07VyziFC8ons/169e79/54/vWuu+6yVatWuXA+XlQEbLdu3fxqvYqACGRJQOOAZglQu4uACIiACIRHAO8kYXg8S4QP6QGfirH9Sy+95DyeCE3yGfHs0SkJL+aCBQtcMXhwsDPOOMNmzZplH374ocsXdQv/+48OUHg4q1Wr5vJB27Zta6tXrw5uovciIAJZEpAHNEuA2l0EREAERCB9AgMGDLB77rkn344XXnihde/e3RB8dAQihJtsXM+WLVu60LofPum+++4zQuoYHZLwgu6xxx5Wu3ZtF2b/6quv7KCDDnLiEoFZtWpVF6IMVgTvbKtWrdy+lEv4nE5HMhEQgfAISICGx1IliYAIiIAIpECAkHYiQ3zSQQhvJflrsaxy5cr29ttvx1oVWdavXz/X+5eculj5iEOHDo1sy5trr7028pk6dOnSxXliw8rvixSuNyIgAiYBqi+BCIiACIhAzhEIq0d2Nh1XyBWV+My5r4YqVEIIKAe0hFxInYYIiIAIiIAIiIAIFBcCEqDF5UqpniIgAiIgAiIgAiJQQggoBB/ChWTcug0bNuzQkzKEopMWwdAgixcvdtsxxhwzdTRo0CDr8dK+/vprNwtIkyZNjBl+SOaPl4uVtJLaQAREQAREQAREQAQCBCRAAzAyfXv//fe7IT8OOeQQ17My03Iy2e+9996z6dOn21FHHeVm5UAsbt682R5++GE3pVwmZbIPoprhTxCgTDHXuHFjCdBMYWo/ERABERABERCBfAQUgs+HI/0PeCCZWaN169bGvMPeGLYjeu7V7777zq92IpFZOBjnzhvbMwDz2rVr3Wtwe7ZhPbN8RBsez5tvvtlNWffggw8ac7POnj3bbcY4eByDMr0xa8PKlSvdDCB+mX9l/DumnEtkCNxg3ZmpItW6JipX60RABERABERABEoH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+umnbebMmbZs2TKbN2+ePfDAA9a1a1ejE11xsjFjxtg999yzg+B75plnbPDgwbZ27Vp3Oq+99poToAjt++67L+Epsg/lpmNr1qyxcePGRY4FU5kIiEB4BCRAw2OpkkRABESgSAm0adPGbr75Zrv11lvtiSeesDp16tiQIUPy1QkP4vr16/Mt8yKV5dHr2DB6n23bttmPP/6YrwzvmWQhHscVK1bk24btf/nlF7fsiy++sN9++y3f/sEPDRo0cGLaL/v999/tgw8+sKpVq/pF1q9fP9trr71s1apVToh6bygbcKx169ZFtvVvOCbHpv5Bw4P6+eefRzyqwXV6LwIiUDAElANaMFxVqgiIgAgUKYGdd97Z2rVrZzfccIMLxSMKBwwY4DyimzdvtgMPPNAGDhxobHfJJZfYYYcd5sQnArRFixZ2/fXXG2HoWPvMnz/fnnrqKRs5cqQ7x8WLF9vdd99tTz75pBO/lLHnnnsayxHDzZs3t8cee8y++eYbJwD32GMP++mnn2zUqFG2++6778DplFNOsSlTptgVV1zh1uHlpH6Ewr1169bN+vfv78LyCGg8pJdeeqkr88UXX3Ti+4cffrB//OMfbhfOv0ePHla2bFlbunSpDRo0yBo3buxSFe644w7Hg+XXXnutIeST2ezZsyMi909/+pOdeuqpcXcpjqkQcU9GK0oVgegHTR4Gg8vKlStnO+2UmS9TArRUfZV0siIgAqWJwEEHHeS8jngDH330UatZs6bziOIJRJgi7I455hiHpHbt2k7QIRI7depk1113nT300EMx92natKnde++99tVXX7n15Jy2bt3avvzySyco8b5ipAW88sorToDyme0nTJhgZcqUceLy/fffd2KXdUGjLrvssovzotatW9def/11J/CCApTtK1as6Or63nvvOfHJesTn5MmTjRsjIhmhuM8++xii+5FHHrHq1avb6NGjXZ4pAnTs2LF222232dFHH+08twjQM888M1idmO8R2z6vFBHdpEmTmNtpoQgUZwI+OhI8h+AyHugkQIN09F4EREAERMCJTzDstttu9umnn1rfvn0dFTx2J510kpFH6QXocccd59bVqFHDtm/f7sLUifY544wz7NVXX7WLL77Y3njjDXv88cetSpUqduGFF9rEiRNt+fLl9sknnxhi0luzZs2c+OTzvvvum8+T4rfxry1btrRZs2bZ/vvvb4sWLbLevXv7VXFfP/vsMycEEZ8YObAY9eB4iE+sXr16zptK2gCiFQGNUMa2bNniWLkPCf49+OCDkTQCBPWuu+4ad2u8RniTZSJQ3AhUq1YtX5X5zZD64o3vfqYmD2im5LSfCIiACOQ4AYRb5cqV3Q0Dcfjrr79Gakw+JsLIW/ny5f3biEhMtA9ewj59+lj9+vXdH9sSmics3rFjRxf+P+KII4ze+d4qVKjg30aOEVkQ9QYBessttxj5oHgXSRVIZoT9g7mliEmf7xkUiMGbJmL87LPPNl6xc88914lVckUTGZ4f/lIxBL1MBIojgWjvJr+d6GWZnldmgftMj6b9REAEREAECpwAnWrwXg4fPtw6d+7sjoegw2OJQCOHC69lw4YNE9Yl0T61atVyU+USzvYhazyNhLIRoAcffLDNmTMnnyBMeLColfvtt58Lo1N+ovxKRKAfhun44493Hk0fIiRHlfB9PMOTAwNSFHjF20POa1CYx9tXy0VABLIjIAGaIT/yiRhzjyT3VM03kmzPjSDYazPVMjLdjmP7MQJ5pVFO9oSf7Fi44n0OVLJttV4ERKDgCeB9PPHEE10+Jp2C8Ob5CR1OO+0027Bhg3Xo0MGJUgQi6xNZsn3orEPOKJ2MsNNPP921K3T2ueqqqwyRyvidmRqdkWhrDz300LhFkCP68ccfO28sXtzzzjvPCWA6JNHxqG3btnH3ZYXvuES4/sYbb3R88BrLREAECpZAmbzQgGIDGTBmfDh6XXbp0iXSwCcqBq8DuVHsg9EokkNEXlJh2KZNm9zN5pxzznGHo1EnPHfyySfbX//614yqwLAl48ePd71cMyogwU70ovX5Wwk20yoRyCkCfiYkwtv85ghL05kml4ywNDmS6dQr3X3wQBJuD4a6C5IBvcxJL/Bhdt7z9+c//znlw0bntqW8Ywob4lFF/NNzXyYCxYmAb9N8nXmoq1Spkv+Y1atyQDPE99JLL7khPRjc2HsYKIrQFrlECE5EFGEkPiP48BREN3Is40bAjSpoNJ4MnoxA9blJNOrkOFEu5pODo28O7EveUzCny5fNE74vj7ATnoojjzzSCVG2oaGkTjTo9Bxl2++//9415H4/bq784d2gJ6036kHd2C+dht/vr1cREIGCJxCrXUh21HT3oZ0qTCMnzYtPjkubmo7AZp9gxwo+y0RABAqWgARoBnwJ95AQTy9QBChDmRx11FGupHhj3bEdzmbCYoy9hzGrB0KVUDhDmDD8B8bQJ0wBxxAjGzdudNsxZl+8sfoYS48G//LLL3f7M/QJqQGIy0RGj1B6wjL13cl5nlDqwZh9DPaM94awPWWTR8aA1uR1YQznggfl2GOPdTOQkKPFtHgsZ7slS5bYNddcY4TvvNE7lZw0b4wzmEikyjHvSem1OBHw4+Pxu8Z4EOSB0Bv5iv5Bzi/TqwiIgAiURgLKAc3gquP9RDBivDJgctD8WHeIUUQWY90hBnlKZ8o4fwMiYZ4wPILzn//8pxOoiEGGHnnyySdtxIgRLh+JMe28MaQJifWMZ/fCCy84TyUdABCA3hhOxHcK8MvivdKDdfXq1W71ggULnMhk5hTGz2OwaMQ1ZZGziuEZZegWf/5uYd4/mFx99dVuCj0ENt7QoJFz+n//93+RP6a5w6Mb708CNEhP74sLAf999vndPAj6ZbwGxWhxOSfVUwREQAQKgoA8oGlSxcOBQCR5n7HuuKngrcRj6BPXUx3rjtA3hqgk9M1NCy8h4/H5cezohco4ezfddJPbNtZYfYcccogbDoScTkQuoveAAw5w2yf7h5eTMQIxzokBnfFkMo0eQpr1jRo1cnXDQ8pYdgxmTZiddAJvCFJ6jzIINMI66P1kGzyiV155pd/chccS5YeRLyUTgeJGwI8zSYoKuVK0CemGgovbOau+IiACIpAJAXlA06Q2c+ZMJ8BIwsVLxw2GXpjTpk2LlJTqWHfBnCUvxvbee+98XhKfb+nH3QrmYvl9OLD3guIJjfZORioW4w2ilfpjw4YNc7OUkLfas2fPSM9TjkP5eD7jeVfp/frcc8854YkXNzr8T8oC5+v/OB/KjfcXo6paJAI5TyD4faaywc/+fc6fhCooAiIgAoVAQAI0TcgITTodMVWd/6N3O0MbJRo7zofdk4XgyMWcO3duxLtIDzQ8nF6AxqsuApAp5/BgJhozz+9PxyLC5gyjxHzRGDOCkOdJbiteG3I5/TmxDM8vIXnqGG39+vVz3lvEL4NH05HJ58FFb6vPIiACIiACIiACpZuAQvBpXH/mOSY0TcedoBFyJreTQZfjGR7Aww8/3P7yl78YQzjFM3rDM7YeQhDvKnmYdFxKZnhk6QCEcEzUA5WQPp5b5i5mOjrqTUgdY8Bq8lHJOcVbQ4oAYXiMHvf80UHJh+zdiv/+o75Dhw51cyzTcYp5pL3oDm6n9yIgAiIgAiIgAiKgcUAL+TtAr1if35no0ImGUoq3H9PikcdJ7/RMjU5GdJwIphGkUxaeVURzMo9tsjI1DmgyQlqfiwT8mHmkzuTqOKC5yK2k14lIksYBLelXuWSen2/T/NlpHFBPohi+piI+OS08mal2Xli8eLE9//zzroNQ06ZNs6KCcMxUfHLgbPbNquLaWQREQAREQAREoNgQUAi+2Fyq+BUlbM9UdfQ0J3QuEwEREAEREAEREIFcJiABmstXJ8W60XOe3FKZCIiACIhA7hJgzOhUo2DJziLMUCjDCTLkHjn+YRhpXETwgiO9ZFouaWGkZNG3IdHkJemUHyY70s5++umnyMyE6dQj1rYMx0ifkbC+J3QIJjJJv49cM/WCz7UrovqIgAiIgAiIgAiIQAknIAFawi+wTk8EREAEREAEREAEco2ABGiuXRHVRwREQAREQAREQARKOAHlgJbwC6zTEwEREAERyA0CDJMnE4GCIBA9XFJBHCPsMuUBDZuoyhMBERABERABERABEUhIQB7QhHi0sigJTJo0KetxRcPo7RhWD1F6SzKLVKrju8ZiH9YA55RDr9fy5cvHOkzKy+idSs/UbMspadcpZYDaUAREQARKKQF5QEvphddpi4AIiIAIiIAIiEBREZAALSryOq4IiIAIiIAIiIAIlFICCsGX0gtfHE67ffv2RV7N4pjYXeTQVAEREAEREAERSEJAHtAkgLRaBERABERABERABEQgXAISoOHyVGkiIAIiIAIiIAIiIAJJCEiAJgGk1SIgAiIgAiIgAiIgAuESkAANl6dKEwEREAEREAEREAERSEJAAjQJIK0WAREQAREQAREQAREIl4B6wYfLU6WJgAiIQIETWL58uS1evNgdZ6eddrLq1atbgwYN3KQAiQ7OgP+ffPKJnXjiiYk2S2nd1q1b7fXXX49sW7FiRatfv75Vq1YtsizbN/PmzbN99tnH9t1332yLSmn/Tz/91MqWLWv16tWLbP/hhx/ajz/+aCeccEJkmd6IgAhkT0Ae0OwZqgQREAERKFQC7733nk2YMMGWLVtmixYtsieffNKuvPJKY7atRIYAffPNNxNtkvK6zZs32/333+/qsHTpUnv11VetW7duNnfu3JTLSLbhCy+8EBHaybYNY/1nn31miHtvzBh23333ORHsl+lVBEQgHALygIbDUaWIgAiIQKESwON58803R4553XXX2ezZs+2MM86ILFuzZo1VqlTJdt99d7esdu3adsMNN0TW//777/bNN9/YH3/84UTWn/70J/vtt99s27ZtbtpY9t97770j+0d2/O+bnXfeOV8dnn/+eeOvadOmkU2/++47Q8hFe0Z//fVXW7t2rdWsWdPwpjKda5kyZdx+/riRQv775qeffnL77LnnnlalShW3FO+krzdTw+63337uc3DfaA4IdaaQZT+M+vF31llnRerAcvjAGG4yERCBcAlIgIbLM25peAveeusta9mype2xxx6R7Rjo/KSTTnKNfWRhFm82bNhg8+fPN+Yvb9asmdWqVSuL0rSrCIhAcSDwww8/GEIPMYe98847NnLkSCMsjpf0qquusrZt2xoevgceeMAee+wx++KLL+zWW2+1qlWr2qZNm+znn3+2UaNG2erVq23IkCFOlCIKV65caf3797ejjz46KQqEHgIQ+/bbb23AgAGuLaL9O/DAA23gwIGGaKUtHDRokFuG+Fy3bp1NnDjRid9rr73WiVHqRDjc29SpU23s2LFuH86DNIJevXq5c0FEcz60rYhUzgPRHY/D8OHDrU6dOtaxY0dX/KOPPmrlypVz+1aoUMEuvvhiW7hwofXr18+F4/H09uzZ04499lhfHb2KgAhkSUACNEuAqe4+bdo0e+aZZ5xnITjDz0MPPWSNGzcORYC+9tprRsN63HHHOa8HjSdl33TTTalWU9uJgAgUEwJvv/220ZbgpUN8nnbaaXbooYc6EXXXXXfZPffc437/eAUvv/xyO/vss/Od2YIFC5wAw+uHsQ35jng8CUOPGTPGDjjgABs/frwh/mIJULyY1GH79u22ZcsW51VkP4y2DUGMmMWriueV8hs1amTUj7aKnNFZs2bZbbfd5vZ5+OGH7fjjj7drrrnGCdfzzjvPLad88kGHDRvmPLUIzg4dOkTatq+++sqlJOBBveKKK+z99993YjEehzPPPNMdHwGK95e2k/rivfWGOP+///s/Vw75ts8991w+AXrjjTc6gcv2iF0EdDyj/jIRKEgCPMTFMzz+/D7DML7LwWMRifCRhHTLlwBNl1iG27/00kvWo0cP16gHBWh0cXgKuJn4MBJeCcJh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jYcTLz4plzaNkDYeWlKBEJY8jLPcG/vEaj+IuATFp98+2WuQQ7Jto9dLfEYT0WcRCIeAPKDhcIxZCnmXeC4Rl0FDENJjHRHojc47hKoQqoSbeMrv06eP0YGJHFLCQDSi559/vhvChA4EwQaahplhV1jOTQbD04o3gMb89NNPdw09OVQIVDwD8WZEwbOC5xSvKd4F3wATWsfLeskllzhPCuUfc8wx7oZAGA1xi4eXENndd9/tT02vIiACOUKATobR7VFRVA3RSerOqlWrXO923yO+KOqiY4qACBQNgTJ54ZDtRXNoHTUWATyXhFizCWlSBuHVWDmLhJPweAS9DbHqgUiljOCg+X478lN9ONkv45V9WJdN3X15eHmDKQp+eWG/4s2FGSwSdZpKpV50FAkjBE/+MB0sgl6lVI4f3IZQbBgheK4TDz/ZXnO896SpZGMl8Tplw0P75g4Bol109GKqUpkIFAQB7lWxjCgl9/xYeiDW9smWhdFW+2PIA+pJ5MgrifjZ3swpI15nB+/NTHa6iJt4AsfnkEWXkWif6G31WQREQAREQAREoPQSUA5o6b32OnMREAEREAEREAERKBICEqBFgl0HFQEREAEREAEREIHSS0ACtPRee525CIiACIiACIiACBQJAQnQIsGug4qACIiACIiACIhA6SUgAVp6r73OXAREQAREQAREQASKhIAEaJFg10FFQAREQAREQAREoPQSkAAtvddeZy4CIiACIiACIiACRUJA44AWCXYdNBUCkyZNynqawDAHzU2lztpGBERABOIRmDJlisUbRznePvGWh9m2hTWJg6/rDz/84MaRzmTaVF+Gf/3jjz/c9NOMYc2kF2FYmOyYYITJY5g5MAzbunWrmwkxrO9JGHUqqDLkAS0osipXBERABERABERABEQgJgEJ0JhYtFAEREAEREAEREAERKCgCEiAFhRZlSsCIiACIiACIiACIhCTgARoTCxaKAIiIAIiIAIiIAIiUFAEJEALiqzKzYrAhRdeaE899VRWZbBzpUqVsi5j5MiRdvnll2ddToUKFVxifjYFffrpp9a6dWv74osvsinGypYta+XLl8+qDHbu3LlzzlynESNG2BVXXJH1OYVxnT755JNQrlPWJ6MCcoLApk2b3Pdhzpw5odUnjLbNV+aRRx6x7t27+49Zv+6xxx4WRgckKkIHH9q8mTNnZl0vX0CY7LhPXXTRRb7orF9333330DqqURnYvfjii1nXyxcQJjsJUE9VrzlFYNu2bfbrr7/mRJ1++eUXoz65YL///rtrkOkZmgum6xT7KnB9uHHmynWKXUstLSwC27dvd9+H3377rbAOmdZxcqmNi6642EUTSe8z7VCufu8kQNO7ltpaBERABERABERABEQgSwIaBzRLgNq9YAgcd9xxduCBBxZM4WmWetBBB9nPP/+c5l4Fs3nFihXt5JNPNkJcuWDNmzfPqeuUK17zXLtOufBdKc11IBzN73bvvffOSQy0cbnqrf/Tn/7k2FWvXj0n2dWpU8e4X+Wq8b2rWbNmTlavTJ57e3tO1kyVEgEREAEREAEREAERKJEEdiqRZ6WTEgEREAEREAEREAERyFkCEqA5e2lUMREQAREQAREQAREomQSUA1oyr2uxPqtvv/3WPvzwQ6tdu7bVr1+/SM9lwYIF+fI/yZUKcxiKVE9uxowZ1qJFCyMfyltRcVqyZIkbJoTr460oOH333Xfue9KwYUOrUaOGr4p7Xbp0qX355Zd25JFHWpUqVfKtK4gP5J7ynd1tt93s8MMPtzJlyrjDMCc2QzIF7dhjjw1+1PtSQKCofqupoC2K324q9YrVzuQKR37XH3/8sZED72316tW2du1a/9EqV65s9erVi3wurDcrVqywNWvWGO0M7ZE3+jF88MEHrm1q0qRJ1kMC+nKzed359jzLpgDtKwJhEqAxvOGGG4xOHIy/Wa5cOTvkkEPCPETKZTF0RdeuXe3f//630Rjyd8ABB1i1atVSLiOMDZ977jkbNGiQdenSJSJAi4oT449yfegg5hvXouD0z3/+0+6//37bc889bcKECUaj64Xd0KFDbcqUKcbQMg899JC7SbBdQdm6devs0ksvdQ0747TyvT3rrLPctXr33XftH//4h23YsCHyHTrttNMKqioqNwcJFNVvNRUURfHbTaVesdqZXOHI0HO33XabLVu2zIK/Zdqjd955x1atWuV+67DlYbQw7a9//avNnz/fdSij3dl///1t3333dUOAdevWzbZu3Wq0SbNmzbLTTz898qBcmHXMdyw6IclEIFcIXHzxxds/+ugjV528G/v2vBv59rwntyKp3vLly7fnCdAiOTYHzfOqbe/bt+/2K6+8cvvxxx+/Pa/hi9SlKDjlibrtf/nLX7Z36tRp+/Tp0yN1KWxOeQ379nbt2m3//PPPXR3yGtXtbdq02f7//t//255349retm3b7Xnjpbp1eeJ0+4ABAyJ1LYg3DzzwwPbHH388UvStt966PW/gZ/c5T4xuf/LJJyPr9Kb0ESiK32qqlAv7t5tKveK1M7nAceXKlds7dOjg2uRevXrlO52OHTtuz4u65FtWmB8WLVq0PW9A/Mgh8wbu337jjTe6z6NHj96e92AeWcc9JW9ShMjnonqjHNB8clwfipIAT4xfffVV5KkRT+Of//xn+/rrr4ukWnmNsxu+4l//+pfhcfvxxx8LtR4MOo9XDy9e0IqKE+GcPKHlPJ8+xEy9CpvTzjvv7GZfwhuN4ZEgJAavPFHqvj877fSfpo0Q/Geffea2K6h/zCCDd9obHnMGf8Zgw5BZTz/9tM2bN8/yGnq/mV5LAYGi+q2mirawf7up1CtWO5MrHLkH/O1vf7O8h/B8p8LyvAdg27hxo40bN87dx/JtUAgfGjRoYKNGjYociXaIthEjQkRb6K0w2kV/rESvEqCJ6GhdoRIgTMk0ZEFxQ+iUH3ZRGCEWcgm3bNniXtu3b2/kIBWWMXbgOeecEwm7++MWFadWrVq5vCbqERRSRcGJ7wnG2IXDhg2zM844w+V6fvPNNy4s71bm/WNaTaZBLEhjWtNddtnFHYLpAnmIYvo7jBv8e++956YlHDNmjOV5Tdxy/SsdBIrqt5oq3aL47SarW6x2Jlc4HnrooXbYYYftcAp5nlHXV4CHTB6ESVN66aWXdtiuIBfw0O1zPuE1duxYy/Mau0OSJkRb6K0w2kV/rESv/+vRkGgrrROBQiCAZ4sfb9B48iUPtCiMecX5wwuLkcSNNzTMeX8zOS9x+g81rkf//v2dGMYrgUWz4fvjG+X/7FVw/6dOneq8H0OGDIlMFPDEE0+4fGZuDmeffbblpTA4gZqrA0MXHJ3SWXL09xEKRdmmRV+FXG3jouuZ6xzpp/DCCy/YXnvt5apet25d47eflxoUfSoF/pn82VtuubWzrTYAAAP4SURBVMXI+fR58dH8CrNdTHTC8oAmoqN1hUqAXoMkSSMsvOH93GefffzHQn0l9B+sS61atQwPW1GbOJlLh7j55putfPnydueddxpeSIyZZoIec95H95AviOuHt2HSpEk2fPhwl/jPMegERU98nw5AHUkrwRshKx0Ecu23Gk09V9u46HrmOsfNmzcbo3J4416xfv36Qp9davHixZaX92k9evRwHSF9fRgJJLpdLKr7qq8TrxKgQRp6X6QEGGKoadOmhicJe+utt9wTpX+qLOzKcfwRI0a4w5LjQ3iVoZCK2sTJ7O9//7sboqtPnz7O6+mvCcOLMOwRw5DwlJ/XGciOOeYYv7pAXvM6ZNnrr79ujzzySL4REgjL4w0lBI9xcyAdoFGjRgVSDxWaewRy7bcaTShX27joeuY6R/ItEX7kfpOeNG3aNDvppJMiD5/R51MQn0kP6927t2sbOXbQTjjhBHv55ZddTijbzZ492xo3bhzcpEjeayrOIsGug8YjgMeIHxEhAzxHDHfB2JtFYd9//70b/oix3UguZ8iN6667rlAbFX/eNCCvvfaayyVkWVFyQvwR2vF5joXNCSGX14vToQnmCz/44IOuAxKNP55IxmtlGJK77rprhzxazzWM1/PPP995O4J1yeulb9dff70bG5SOAYwTyncIwRwcOzCM46uM3CZQlL/VZGQK+7ebrD7B9dHtTC5xfOONN4wHz3vvvTdS5aeeesq10Tz4kmNJZKZq1aqR9QX95uGHH3ZD0gXbIdpAhqSjTnfccYd7OOe+mtdj3y644IKCrlLS8iVAkyLSBkVBgJAGY4HmguH9RBDTKSjXTJxiXxEEH+kT9EDPBcNDwk0peHPIhXqpDoVHIJd+q9FnncttXHRdc5kjnSIZkSPY4Se6/kX5mQ615MTjUc4FkwDNhaugOoiACIiACIiACIhAKSKgHNBSdLF1qiIgAiIgAiIgAiKQCwQkQHPhKqgOIiACIiACIiACIlCKCEiAlqKLrVMVAREQAREQAREQgVwgIAGaC1dBdRABERABERABERCBUkRAArQUXWydqgiIgAiIgAiIgAjkAgEJ0Fy4CqqDCIiACIiACIiACJQiAhKgpehi61RFQAREQAREQAREIBcISIDmwlVQHURABERABERABESgFBGQAC1FF1unKgIiIAIiIAIiIAK5QEACNBeuguogAiIgAiIgAiIgAqWIgARoKbrYOlUREAEREAEREAERyAUCEqC5cBVUBxEQAREQAREQAREoRQQkQEvRxdapioAIiIAIiIAIiEAuEJAAzYWroDqIgAiIgAiIgAiIQCkiIAFaii62TlUEREAEREAEREAEcoHA/weyM76fZcVSAwAAAABJRU5ErkJggg==" /><!-- --></p>
</div>
</section>



<!-- dynamically load mathjax for compatibility with self-contained -->
<script>
  (function () {
    var script = document.createElement("script");
    script.type = "text/javascript";
    script.src  = "https://mathjax.rstudio.com/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML";
    document.getElementsByTagName("head")[0].appendChild(script);
  })();
</script>

</body>
</html>



