/**
 * CECD theme, navbar + timeline interactions.
 * Replaces the React useState bits (Navbar dropdowns/mobile menu, History page scroll timeline).
 */
(function () {
	'use strict';

	document.addEventListener('DOMContentLoaded', function () {
		initNavState();
		initDropdowns();
		initMobileMenu();
		initHistoryTimeline();
	});

	/* Overlay nav: transparent while the viewport is inside the hero, solid
	 * white once the hero scrolls past. Threshold = hero height minus the bar,
	 * so the flip lands exactly where the dark image ends and content begins. */
	function initNavState() {
		var nav = document.querySelector('.cecd-navbar--overlay');
		if (!nav) return;

		var hero = document.querySelector('main.cecd-main--under-nav > section:first-of-type');
		var limit = 64;

		function measure() {
			limit = hero ? Math.max(64, hero.offsetHeight - 96) : 64;
		}
		function onScroll() {
			nav.classList.toggle('is-scrolled', window.scrollY > limit);
		}

		measure();
		onScroll();
		window.addEventListener('scroll', onScroll, { passive: true });
		window.addEventListener('resize', function () { measure(); onScroll(); }, { passive: true });
		// Hero images can change height as they load; re-measure once settled.
		window.addEventListener('load', function () { measure(); onScroll(); });
	}

	function initDropdowns() {
		document.querySelectorAll('[data-cecd-dropdown]').forEach(function (el) {
			var menu = el.querySelector('[data-cecd-dropdown-menu]');
			if (!menu) return;
			el.addEventListener('mouseenter', function () { el.classList.add('is-open'); });
			el.addEventListener('mouseleave', function () { el.classList.remove('is-open'); });
		});
	}

	function initMobileMenu() {
		var toggle = document.querySelector('[data-cecd-mobile-toggle]');
		var menu = document.querySelector('[data-cecd-mobile-menu]');
		if (!toggle || !menu) return;

		var iconOpen = toggle.querySelector('[data-cecd-icon-open]');
		var iconClose = toggle.querySelector('[data-cecd-icon-close]');

		var nav = document.querySelector('.cecd-navbar');

		function setOpen(open) {
			menu.classList.toggle('is-open', open);
			// Overlay nav must go solid while the menu is open, white links on a
			// transparent bar would sit over the hero image behind the panel.
			if (nav) { nav.classList.toggle('is-menu-open', open); }
			if (iconOpen) { iconOpen.classList.toggle('hidden', open); iconOpen.classList.toggle('block', !open); }
			if (iconClose) { iconClose.classList.toggle('hidden', !open); iconClose.classList.toggle('block', open); }
		}

		toggle.addEventListener('click', function () {
			setOpen(!menu.classList.contains('is-open'));
		});
		menu.querySelectorAll('a').forEach(function (a) {
			a.addEventListener('click', function () { setOpen(false); });
		});

		// Accordion: tap a parent group to expand its children. Only one open at a time.
		menu.querySelectorAll('[data-cecd-mobile-group-toggle]').forEach(function (btn) {
			btn.addEventListener('click', function (e) {
				e.preventDefault();
				var group = btn.closest('[data-cecd-mobile-group]');
				if (!group) return;
				var isOpen = group.classList.contains('is-open');

				menu.querySelectorAll('[data-cecd-mobile-group].is-open').forEach(function (g) {
					g.classList.remove('is-open');
				});

				if (!isOpen) {
					group.classList.add('is-open');
				}
			});
		});
	}

	function initHistoryTimeline() {
		// Skip inside Gutenberg's editor iframe, auto-scroll would keep running in the editor preview.
		try { if (window.self !== window.top) return; } catch (e) { return; }
		var scrollEl = document.querySelector('[data-cecd-timeline-scroll]');
		if (!scrollEl) return;

		// Must match $col_w / $pad in blocks/timeline/render.php.
		var COL_W = 290;
		var PAD = 60;
		var AUTO_PX_PER_SEC = 55;

		var events = Array.from(scrollEl.querySelectorAll('[data-cecd-event]'));
		var seenDot = new Set();
		var seenCard = new Set();
		var hoverPaused = false;

		var dragging = false;
		var startX = 0;
		var startScroll = 0;
		var autoPos = scrollEl.scrollLeft;
		var lastT = 0;

		function dotAt(i, cw) { return i === 0 ? 0 : Math.max(0, PAD + i * COL_W + 80 - cw); }
		function cardAt(i, cw) { return i === 0 ? 0 : dotAt(i, cw) + 120; }

		function revealYear(year, type) {
			scrollEl.querySelectorAll('[data-cecd-' + type + '-year="' + year + '"]').forEach(function (n) {
				n.classList.add('is-visible');
			});
		}

		function setHoverState(year, on) {
			scrollEl.querySelectorAll('[data-cecd-dot-year="' + year + '"]').forEach(function (n) {
				n.classList.toggle('is-hovered', on);
			});
			scrollEl.querySelectorAll('[data-cecd-card-year="' + year + '"]').forEach(function (n) {
				n.classList.toggle('is-hovered', on);
			});
		}

		function updateVisibility() {
			var sl = scrollEl.scrollLeft;
			var cw = scrollEl.clientWidth;
			events.forEach(function (ev, i) {
				var year = ev.getAttribute('data-cecd-year');
				if (!seenDot.has(year) && sl >= dotAt(i, cw)) {
					seenDot.add(year);
					revealYear(year, 'dot');
					revealYear(year, 'yearlabel');
				}
				if (!seenCard.has(year) && sl >= cardAt(i, cw)) {
					seenCard.add(year);
					revealYear(year, 'card');
				}
			});
		}

		scrollEl.addEventListener('scroll', updateVisibility, { passive: true });
		updateVisibility();

		// Hover-on-dot (or card) reveals card immediately and pauses auto-scroll.
		function bindHover(selector) {
			scrollEl.querySelectorAll(selector).forEach(function (node) {
				var year = node.getAttribute(selector.replace('[', '').replace(']', '').replace('data-cecd-', 'data-cecd-').replace('=""', ''))
					|| node.dataset.cecdDotYear || node.dataset.cecdCardYear;
				node.addEventListener('mouseenter', function () {
					var y = node.getAttribute('data-cecd-dot-year') || node.getAttribute('data-cecd-card-year');
					if (!y) return;
					if (!seenDot.has(y)) { seenDot.add(y); revealYear(y, 'dot'); revealYear(y, 'yearlabel'); }
					if (!seenCard.has(y)) { seenCard.add(y); revealYear(y, 'card'); }
					setHoverState(y, true);
				});
				node.addEventListener('mouseleave', function () {
					var y = node.getAttribute('data-cecd-dot-year') || node.getAttribute('data-cecd-card-year');
					if (!y) return;
					setHoverState(y, false);
				});
			});
		}
		bindHover('[data-cecd-dot-year]');
		bindHover('[data-cecd-card-year]');

		scrollEl.addEventListener('mousedown', function (e) {
			dragging = true;
			startX = e.clientX;
			startScroll = scrollEl.scrollLeft;
			scrollEl.style.cursor = 'grabbing';
			lastT = 0;
		});
		scrollEl.addEventListener('mousemove', function (e) {
			if (!dragging) return;
			e.preventDefault();
			scrollEl.scrollLeft = startScroll - (e.clientX - startX);
		});
		['mouseup', 'mouseleave'].forEach(function (evt) {
			scrollEl.addEventListener(evt, function () {
				if (dragging) {
					dragging = false;
					scrollEl.style.cursor = 'grab';
				}
			});
		});

		function tick(t) {
			var max = scrollEl.scrollWidth - scrollEl.clientWidth;
			if (dragging || hoverPaused || max <= 0) {
				lastT = 0;
				autoPos = scrollEl.scrollLeft;
				requestAnimationFrame(tick);
				return;
			}
			var dt = lastT ? (t - lastT) / 1000 : 0;
			lastT = t;
			if (Math.abs(scrollEl.scrollLeft - autoPos) > 2) autoPos = scrollEl.scrollLeft;
			autoPos += AUTO_PX_PER_SEC * dt;
			if (autoPos >= max) autoPos = 0;
			scrollEl.scrollLeft = autoPos;
			requestAnimationFrame(tick);
		}
		requestAnimationFrame(tick);
	}
})();
