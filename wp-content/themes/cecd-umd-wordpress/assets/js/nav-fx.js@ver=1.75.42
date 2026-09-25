/**
 * CECD theme, navigation responsiveness layer.
 *
 * 1. Hover/touch prefetch of same-origin links (instant.page-style, ~1KB inlined).
 * 2. Cross-document View Transitions are opted-in via CSS (@view-transition).
 *    Browsers without support fall back to normal navigation, no JS shim.
 *
 * No external dependencies, no plugin requirement.
 */
(function () {
	'use strict';

	var conn = navigator.connection || {};
	if (conn.saveData) return;
	if (/2g/.test(conn.effectiveType || '')) return;

	var prefetched = new Set();
	var hoverDelay = 65;
	var hoverTimer = 0;

	function sameOrigin(href) {
		try {
			var u = new URL(href, location.href);
			if (u.origin !== location.origin) return false;
			if (u.protocol !== 'http:' && u.protocol !== 'https:') return false;
			if (u.hash && u.pathname === location.pathname && u.search === location.search) return false;
			return true;
		} catch (e) { return false; }
	}

	function isPrefetchable(a) {
		if (!a || !a.href) return false;
		if (a.target && a.target !== '' && a.target !== '_self') return false;
		if (a.hasAttribute('download')) return false;
		if (a.dataset.noPrefetch === 'true') return false;
		var rel = (a.rel || '').toLowerCase();
		if (rel.indexOf('external') !== -1 || rel.indexOf('nofollow') !== -1) return false;
		// Skip wp-admin, wp-login, REST API, file downloads.
		if (/\/(wp-admin|wp-login\.php|wp-json)(\/|$|\?)/.test(a.pathname)) return false;
		if (/\.(zip|pdf|docx?|xlsx?|pptx?|csv|tar|gz|mp4|mp3|jpg|jpeg|png|gif|webp|svg)(\?|$)/i.test(a.pathname)) return false;
		return sameOrigin(a.href) && !prefetched.has(a.href);
	}

	function prefetch(href) {
		if (prefetched.has(href)) return;
		prefetched.add(href);
		var link = document.createElement('link');
		link.rel = 'prefetch';
		link.href = href;
		link.as = 'document';
		document.head.appendChild(link);
	}

	document.addEventListener('mouseover', function (e) {
		var a = e.target.closest && e.target.closest('a');
		if (!isPrefetchable(a)) return;
		clearTimeout(hoverTimer);
		hoverTimer = setTimeout(function () { prefetch(a.href); }, hoverDelay);
	}, { passive: true });

	document.addEventListener('mouseout', function () {
		clearTimeout(hoverTimer);
	}, { passive: true });

	document.addEventListener('touchstart', function (e) {
		var a = e.target.closest && e.target.closest('a');
		if (isPrefetchable(a)) prefetch(a.href);
	}, { passive: true });

	/* Image fade-in: stamp [data-cecd-loaded] once a lazy image has decoded. */
	function markLoaded(img) {
		if (img.dataset.cecdLoaded) return;
		img.dataset.cecdLoaded = '1';
	}
	function bindImage(img) {
		if (img.complete && img.naturalWidth > 0) { markLoaded(img); return; }
		img.addEventListener('load', function () { markLoaded(img); }, { once: true });
		img.addEventListener('error', function () { markLoaded(img); }, { once: true });
	}
	document.querySelectorAll('img[loading="lazy"]').forEach(bindImage);

	/* Reveal-on-scroll: any element with data-cecd-reveal fades up when entering view.
	 * Opt-in only (template-side) to avoid flash-of-hidden-content on load. */
	if ('IntersectionObserver' in window && !matchMedia('(prefers-reduced-motion: reduce)').matches) {
		var io = new IntersectionObserver(function (entries) {
			entries.forEach(function (entry) {
				if (entry.isIntersecting) {
					entry.target.classList.add('is-revealed');
					io.unobserve(entry.target);
				}
			});
		}, { rootMargin: '0px 0px -8% 0px', threshold: 0.05 });
		document.querySelectorAll('[data-cecd-reveal]').forEach(function (el) { io.observe(el); });
	} else {
		document.querySelectorAll('[data-cecd-reveal]').forEach(function (el) {
			el.classList.add('is-revealed');
		});
	}
})();
