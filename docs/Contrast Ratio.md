---
layout: default
title: Contrast Ratio 
nav_order: 9
---

# Contrast Ratio tool

<div class="contrast-ratio-tool">
<script>
    (function(){

    var _ = self.Color = function(rgba) {
	if (rgba === 'transparent') {
		rgba = [0,0,0,0];
	}
	else if (typeof rgba === 'string') {
		var rgbaString = rgba;
		rgba = rgbaString.match(/rgba?\(([\d.]+), ([\d.]+), ([\d.]+)(?:, ([\d.]+))?\)/);

		if (rgba) {
			rgba.shift();
		}
		else {
			throw new Error('Invalid string: ' + rgbaString);
		}
	}

	if (rgba[3] === undefined) {
		rgba[3] = 1;
	}

	rgba = rgba.map(a => +a);

	this.rgba = rgba;
};

_.prototype = {
	get rgb () {
		return this.rgba.slice(0,3);
	},

	get alpha () {
		return this.rgba[3];
	},

	set alpha (alpha) {
		this.rgba[3] = alpha;
	},

	get luminance () {
		// Formula: http://www.w3.org/TR/2008/REC-WCAG20-20081211/#relativeluminancedef
		var rgba = this.rgba.slice();

		for(var i=0; i<3; i++) {
			var rgb = rgba[i];

			rgb /= 255;

			rgb = rgb < .03928 ? rgb / 12.92 : Math.pow((rgb + .055) / 1.055, 2.4);

			rgba[i] = rgb;
		}

		return .2126 * rgba[0] + .7152 * rgba[1] + 0.0722 * rgba[2];
	},

	get inverse () {
		return new _([
			255 - this.rgba[0],
			255 - this.rgba[1],
			255 - this.rgba[2],
			this.alpha
		]);
	},

	toString: function() {
		return 'rgb' + (this.alpha < 1? 'a' : '') + '(' + this.rgba.slice(0, this.alpha >= 1? 3 : 4).join(', ') + ')';
	},

	/**
	 * @param {boolean} withAlpha If the output should include the alpha channel.
	 * @returns {string} A hex color string in the format `#RRGGBB` or `#RRGGBBAA.
	 */
	toHex: function(withAlpha = true) {
		var [ r, g, b, a ] = this.rgba;
		var uint8ToHex = function(uint8) { return uint8.toString(16).padStart(2, '0'); }

		var result = `#${uint8ToHex(r)}${uint8ToHex(g)}${uint8ToHex(b)}`;

		if (withAlpha) {
			var aHex = uint8ToHex(a * 255);
			result += aHex;
		}

		return result;
	},

	clone: function() {
		return new _(this.rgba);
	},

	// Overlay a color over another
	overlayOn: function (color) {
		var overlaid = this.clone();

		var alpha = this.alpha;

		if (alpha >= 1) {
			return overlaid;
		}

		for(var i=0; i<3; i++) {
			overlaid.rgba[i] = overlaid.rgba[i] * alpha + color.rgba[i] * color.rgba[3] * (1 - alpha);
		}

		overlaid.rgba[3] = alpha + color.rgba[3] * (1 - alpha);

		return overlaid;
	},

	contrast: function (color) {
		// Formula: http://www.w3.org/TR/2008/REC-WCAG20-20081211/#contrast-ratiodef
		var alpha = this.alpha;

		if (alpha >= 1) {
			if (color.alpha < 1) {
				color = color.overlayOn(this);
			}

			var l1 = this.luminance + .05,
				l2 = color.luminance + .05,
				ratio = l1/l2;

			if (l2 > l1) {
				ratio = 1 / ratio;
			}

			// ratio = floor(ratio, 2);

			return {
				ratio: ratio,
				error: 0,
				min: ratio,
				max: ratio
			};
		}

		// If we’re here, it means we have a semi-transparent background
		// The text color may or may not be semi-transparent, but that doesn't matter

		var onBlack = this.overlayOn(_.BLACK),
		    onWhite = this.overlayOn(_.WHITE),
		    contrastOnBlack = onBlack.contrast(color).ratio,
		    contrastOnWhite = onWhite.contrast(color).ratio;

		var max = Math.max(contrastOnBlack, contrastOnWhite);

		// This is here for backwards compatibility and not used to calculate
		// `min`.  Note that there may be other colors with a closer luminance to
		// `color` if they have a different hue than `this`.
		var closest = this.rgb.map(function(c, i) {
			return Math.min(Math.max(0, (color.rgb[i] - c * alpha)/(1-alpha)), 255);
		});

		closest = new _(closest);

		var min = 1;
		if (onBlack.luminance > color.luminance) {
			min = contrastOnBlack;
		}
		else if (onWhite.luminance < color.luminance) {
			min = contrastOnWhite;
		}

		return {
			ratio: (min + max) / 2,
			error: (max - min) / 2,
			min: min,
			max: max,
			closest: closest,
			farthest: onWhite == max? _.WHITE : _.BLACK
		};
	}
};

_.BLACK = new _([0,0,0]);
_.GRAY = new _([127.5, 127.5, 127.5]);
_.WHITE = new _([255,255,255]);

})();
function $(expr, con) {
	return typeof expr === "string"? (con || document).querySelector(expr) : expr;
}

function $$(expr, con) {
	return Array.prototype.slice.call((con || document).querySelectorAll(expr));
}

/*
 * Make each element with an ID a global variable.
 * Many browsers do this anyway (it’s in the HTML5 spec), so it ensures consistency.
 *
 * https://html.spec.whatwg.org/multipage/window-object.html#named-access-on-the-window-object
 */
$$("[id]").forEach(function(element) {
	window[element.id] = element;
});

// Math.floor with precision
function floor(number, decimals) {
	decimals = +decimals || 0;

	var multiplier = Math.pow(10, decimals);

	return Math.floor(number * multiplier) / multiplier;
}

var messages = {
	"semitransparent": "The background is semi-transparent, so the contrast ratio cannot be precise. Depending on what’s going to be underneath, it could be any of the following:",
	"fail": "Fails WCAG 2.0 and 2.1 :-(",
	"aa-large": "Passes AA for large text (above 18pt or bold above 14pt) and AA for user interface components and graphical objects",
	"aa": "Passes AA level for any size text, AAA for large text (above 18pt or bold above 14pt), and AA for user interface components and graphical objects",
	"aaa": "Passes AAA level for any size text and AA for user interface components and graphical objects"
};

var canvas = document.createElement("canvas"),
    ctx = canvas.getContext("2d");

canvas.width = canvas.height = 16;
document.body.appendChild(canvas);

incrementable.onload = function() {
	if (window.Incrementable) {
		new Incrementable(background);
		new Incrementable(foreground);
	}
};

if (window.Incrementable) {
	incrementable.onload();
}

var output = $(".contrast");

var levels = {
	"fail": {
		range: [0, 3],
		color: "hsl(0, 100%, 40%)"
	},
	"aa-large": {
		range: [3, 4.5],
		color: "hsl(40, 100%, 45%)"
	},
	"aa": {
		range: [4.5, 7],
		color: "hsl(80, 60%, 45%)"
	},
	"aaa": {
		range: [7, 22],
		color: "hsl(95, 60%, 41%)"
	}
};

function rangeIntersect(min, max, upper, lower) {
	return (max < upper? max : upper) - (lower < min? min : lower);
}

function updateLuminance(input) {
	var luminanceOutput = $(".rl", input.parentNode.parentNode);

	var color = input.color;

	if (input.color.alpha < 1) {
		var lumBlack = color.overlayOn(Color.BLACK).luminance;
		var lumWhite = color.overlayOn(Color.WHITE).luminance;

		luminanceOutput.textContent = lumBlack + " - " + lumWhite;
		luminanceOutput.style.color = Math.min(lumBlack, lumWhite) < .2? "white" : "black";
	}
	else {
		luminanceOutput.textContent = color.luminance;
		luminanceOutput.style.color = color.luminance < .2? "white" : "black";
	}
}

function update() {
	if (foreground.color && background.color) {
		if (foreground.value !== foreground.defaultValue || background.value !== background.defaultValue) {
			window.onhashchange = null;

			location.hash = "#" + encodeURIComponent(foreground.value) + "-on-" + encodeURIComponent(background.value);

			setTimeout(function() {
				window.onhashchange = hashchange;
			}, 10);
		}

		var contrast = background.color.contrast(foreground.color);

		updateLuminance(background);
		updateLuminance(foreground);

		var min = contrast.min,
		    max = contrast.max,
		    range = max - min,
		    classes = [], percentages = [];

		for (var level in levels) {
			var bounds = levels[level].range,
			    lower = bounds[0],
			    upper = bounds[1];

			if (min < upper && max >= lower) {
				classes.push(level);

				percentages.push({
					level: level,
					percentage: 100 * rangeIntersect(min, max, upper, lower) / range
				});
			}
		}

		$("strong", output).textContent = floor(contrast.ratio, 2);

		preciseContrast.innerHTML = `Precise contrast: ${contrast.ratio - contrast.error}`;

		var error = $(".error", output);

		if (contrast.error) {
			error.textContent = "±" + floor(contrast.error, 2);
			error.title = floor(min, 2) + " - " + floor(max, 2);
			preciseContrast.textContent = `${min} - ${max}`;
		}
		else {
			error.textContent = "";
			error.title = "";
			preciseContrast.textContent = contrast.ratio;
		}

		if (classes.length <= 1) {
			wcag.textContent = messages[classes[0]];
			output.style.backgroundImage = "";
			output.style.backgroundColor = levels[classes[0]].color;
		}
		else {
			var fragment = document.createDocumentFragment();

			var p = document.createElement("p");
			p.textContent = messages.semitransparent;
			fragment.appendChild(p);

			var ul = document.createElement("ul");

			for (var i=0; i<classes.length; i++) {
				var li = document.createElement("li");

				li.textContent = messages[classes[i]];

				ul.appendChild(li);
			}

			fragment.appendChild(ul);

			wcag.textContent = "";
			wcag.appendChild(fragment);

			// Create gradient illustrating levels
			var stops = [], previousPercentage = 0;

			for (var i=0; i < 2 * percentages.length; i++) {
				var info = percentages[i % percentages.length];

				var level = info.level;
				var color = levels[level].color,
				    percentage = previousPercentage + info.percentage / 2;

				stops.push(color + " " + previousPercentage + "%", color + " " + percentage + "%");

				previousPercentage = percentage;
			}

			var gradient = "linear-gradient(135deg, " + stops.join(", ") + ")";

			output.style.backgroundImage = gradient;
		}

		output.className = "contrast " + classes.join(" ");

		ctx.clearRect(0, 0, 16, 16);

		ctx.fillStyle = background.color + "";
		ctx.fillRect(0, 0, 8, 16);

		ctx.fillStyle = foreground.color + "";
		ctx.fillRect(8, 0, 8, 16);

		$("link[rel=\"shortcut icon\"]").setAttribute("href", canvas.toDataURL());
	}
}

function colorChanged(input) {
	input.style.width = input.value.length * .56 + "em";
	input.style.width = input.value.length + "ch";

	var isForeground = input == foreground;

	var display = isForeground? foregroundDisplay : backgroundDisplay;

	var previousColor = getComputedStyle(display).backgroundColor;

	// Add hash to front of 3, 4, 6, and 8 digit hex codes.
	var accepted_matches = [3, 4, 6, 8]
	var match_result = input.value.match(/^[0-9a-f]{3,8}$/i)
	if (match_result && accepted_matches.includes(match_result[0].length)){
		input.value = "#" + input.value;
	}

	display.style.background = input.value;

	var color = getComputedStyle(display).backgroundColor;

	if (color && input.value && (color !== previousColor || color === "transparent" || color === "rgba(0, 0, 0, 0)")) {
		// Valid & different color
		if (isForeground) {
			backgroundDisplay.style.color = input.value;
		}

		input.color = new Color(color);

		return true;
	}

	return false;
}

function hashchange() {

	if (location.hash) {
		var colors = location.hash.slice(1).split("-on-");

		foreground.value = decodeURIComponent(colors[0]);
		background.value = decodeURIComponent(colors[1]);
	}
	else {
		foreground.value = foreground.defaultValue;
		background.value = background.defaultValue;
	}

	background.oninput();
	foreground.oninput();
}

background.oninput =
foreground.oninput = function() {
	var valid = colorChanged(this);

	if (!valid) {
		return;
	}

	update();

	if (this === background) {
		var bgStyle = getComputedStyle(backgroundDisplay).backgroundColor;
		backgroundColorPicker.value = new Color(bgStyle).toHex(false);
	}
	else {
		var fgStyle = getComputedStyle(foregroundDisplay).backgroundColor;
		foregroundColorPicker.value = new Color(fgStyle).toHex(false);
	}
};

backgroundColorPicker.oninput = (event) => {
	background.value = event.target.value;
	colorChanged(background);
	update();
};

foregroundColorPicker.oninput = function(event) {
	foreground.value = event.target.value;
	colorChanged(foreground);
	update();
};

swap.onclick = function() {
	var backgroundColor = background.value;
	background.value = foreground.value;
	foreground.value = backgroundColor;

	colorChanged(background);
	colorChanged(foreground);

	update();

	var bgStyle = getComputedStyle(backgroundDisplay).backgroundColor;
	backgroundColorPicker.value = new Color(bgStyle).toHex(false);

	var fgStyle = getComputedStyle(foregroundDisplay).backgroundColor;
	foregroundColorPicker.value = new Color(fgStyle).toHex(false);
};

window.encodeURIComponent = (function(){
	var encodeURIComponent = window.encodeURIComponent;

	return function (str) {
		return encodeURIComponent(str).replace(/[()]/g, function ($0) {
			return escape($0);
		});
	};
})();

window.decodeURIComponent = (function(){
	var decodeURIComponent = window.decodeURIComponent;

	return function (str) {
		return str.search(/%[\da-f]/i) > -1? decodeURIComponent(str) : str;
	};
})();

(onhashchange = hashchange)();
</script>

<h1><a href="#"><div><span>Contrast</span></div> <div><strong>ratio</strong></div></a></h1>

<label class="background">
	<span>Background:</span>
	<div class="input-wrapper">
		<input id="background" class="text-input" value="white" autofocus />
		<input id="backgroundColorPicker" class="color-picker" type="color" tabindex="-1" />
	</div>
	<output for="background foreground" class="rl" aria-live="polite" aria-label="Relative Luminance"></output>
</label>

<label class="foreground">
	<span>Text color:</span>
	<div class="input-wrapper">
		<input id="foreground" class="text-input" value="hsla(200,0%,0%,.7)" />
		<input id="foregroundColorPicker" class="color-picker" type="color" tabindex="-1" />
	</div>
	<output for="background foreground" class="rl" aria-live="polite" aria-label="Relative Luminance"></output>
</label>

<output for="background foreground" class="contrast" tabindex="0" aria-live="polite">
	<strong>?</strong>
	<span class="error"></span>
</output>

<section id="results">
	<output for="background foreground" id="preciseContrast" aria-label="Precise contrast"></output>
	<output for="background foreground" id="wcag"></output>
</section>

<section class="color-display" id="backgroundDisplay">
	<h1>How to use</h1>
	<p>As you type, the contrast ratio indicated will update. Hover over the circle to get more detailed information. When semi-transparent colors are involved as backgrounds, the contrast ratio will have an error margin, to account for the different colors they may be over.</p>
	<p style="font-family: Garamond, 'Palatino Linotype', Georgia, serif;">This sample text attempts to visually demonstrate how readable this color combination is, for normal, <span style="font-style: italic;">italic</span>, <span style="font-weight: bold;">bold</span>, or <span style="font-style: italic; font-weight: bold;">bold italic</span> text of various sizes and font styles.</p>
	<p style="font-size: 11pt;"><strong>Hint:</strong> Press the up and down keyboard arrows while over a number inside a functional color notation. Watch it increment/decrement. Try with the Shift or Alt key too!</p>
	<footer>
		By <a href="http://lea.verou.me" rel="noopener" target="_blank">Lea Verou</a>
		&bull; <a href="https://www.w3.org/TR/WCAG/#contrast-minimum" rel="noopener" target="_blank">WCAG 2.1 on contrast ratio</a>

	</footer>
</section>
<section class="color-display" id="foregroundDisplay">
	<button id="swap">↔ Swap colors</button>
	<script async src="https://cdn.carbonads.com/carbon.js?zoneid=1673&serve=C6AILKT&placement=contrastratiocom" id="_carbonads_js"></script>
</section>

<footer></footer>

<script src="https://leaverou.github.io/incrementable/incrementable.js" id="incrementable" async></script>
<script src="color.js"></script>
<script src="contrast-ratio.js"></script>

<div class="social">
	<a class="github-button" href="https://github.com/leaverou/contrast-ratio" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star leaverou/contrast-ratio on GitHub">Star</a>
</div>

<!-- Place this tag in your head or just before your close div tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>

<script>
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

ga('create', 'UA-117109922-1', 'auto');
ga('send', 'pageview');
</script>
</div>