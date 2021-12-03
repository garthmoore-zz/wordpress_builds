## Hosting with WPEngine
- Global Edge Security (GES) (or Cloudflare)
- Smart Plugin Manager
- Let’s Encrypt SSL
- WPE supports beta and dev environments, which can be 1:1 clones of production, including caching, CDN, etc. Be careful when copying the beta site to production, since this can break URLs and assets on the live site. I recommend trying only testing code in beta and adding content to the live site directly instead. If you do have to overwrite production with beta, make sure to use Better Search and Replace to make sure the live site still only uses URLs that are on the correct domain, SSL's, and www/non-www.

## Deployments

Regular developers and (especially) site admins/editors should *not* have FTP access to the site. All site code changes should be pushed via [git deployment](https://wpengine.com/support/git/).

Only put the child theme folder into the WPE git repo (/wp-content/themes/child-theme/). Don't include the WP core, plugins, or parent themes, per WPE's recommendations. Include built assets, such as minified CSS/JS, since WPE doesn't have a build pipeline.

## Emails

Add [Postmark](https://postmarkapp.com/) on all WP installs to ensure deliverability of transactional emails like password resets, form notifications, etc.

Postmark has several agency/developer-friendly features:

- Combined billing for multiple clients ($10/mo for 10k emails)
- Easy and centralized sending domain management
- Delegated users
- 1st party WordPress plugin
- Support for client-specific API keys
- Deliverability reports

## Plugins

### Security
-	[Forbid Pwned Passwords](https://wordpress.org/plugins/forbid-pwned-passwords/)
-	[Disable REST API](https://wordpress.org/plugins/disable-json-api/)
-	[Disable Emojis](https://wordpress.org/plugins/disable-emojis/)
-	[Limit Login Attempts Reloaded](https://wordpress.org/plugins/limit-login-attempts-reloaded/)
-	[wp-password-bcrypt](https://github.com/roots/wp-password-bcrypt)
	-	This is a MU plugin that has to be installed via (S)FTP. It's an in-place upgrade for WP's default password hashing that migrates users to bcrypt on their next login.

### Configuration
- [Advanced Custom Fields Pro](https://www.advancedcustomfields.com/pro/)
	- Make sure ACF is configured for [local JSON](https://www.advancedcustomfields.com/resources/local-json/), so that the configuration gets saved to the theme, rather than the WP DB.
- [Better Search and Replace](https://wordpress.org/plugins/better-search-replace/)
- [Classic Editor](https://wordpress.org/plugins/classic-editor/)
- [Disable Blog](https://wordpress.org/plugins/disable-blog/), only use for sites without posts
- [Disable Comments](https://wordpress.org/plugins/disable-comments/)
- [Disable Embeds](https://wordpress.org/plugins/disable-embeds/)
- [Disable Emojis](https://wordpress.org/plugins/disable-emojis/)
- [Enable Media Replace](https://wordpress.org/plugins/enable-media-replace/)
- [EWWW Image Optimizer](https://wordpress.org/plugins/ewww-image-optimizer/)
- [GDPR Cookie Compliance](https://wordpress.org/plugins/gdpr-cookie-compliance/)
-	[Imsanity](https://wordpress.org/plugins/imsanity/)
-	[Yoast Duplicate Post](https://wordpress.org/plugins/duplicate-post/)

### SEO
- Yoast SEO [free](https://wordpress.org/plugins/wordpress-seo/) or [premium](https://yoast.com/wordpress/plugins/seo/) (paid annual $89)
    - Also consider whether the [local](https://yoast.com/wordpress/plugins/local-seo/), [news](https://yoast.com/wordpress/plugins/news-seo/), [video](https://yoast.com/wordpress/plugins/video-seo/), and/or [eCommerce](https://yoast.com/wordpress/plugins/yoast-woocommerce-seo/) paid extensions are appropriate.
-	[Redirection](https://wordpress.org/plugins/redirection/)
	-	Only if you're not using Yoast SEO Premium, which has its own redirection features.

### Forms
- [Gravity Forms](https://www.gravityforms.com/) (paid annual $159)

### Site Search
-	Relevanssi [free product](https://wordpress.org/plugins/relevanssi/) or [Premium](https://www.relevanssi.com/buy-premium/) (paid annual $99)

## Themes

I recommend [Genesis](https://www.studiopress.com/themes/genesis/) with my [starter child theme](https://github.com/cdukes/bones-for-genesis-2-0).

### Setup, Security, & QA

Review the HTML <head> to make sure that all elements present are sensible, and that nothing important is missing. For instance, every page should have a <title> and meta description, but the RSS feed can be left out if the site won't be publishing regular articles.

Add the [Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security) header whenever possible: `Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`. **Before using this, make sure that all URLs on the domain and subdomains are using SSL. Once HSTS is active, the entire domain will irreversibly refuse to load non-SSL content.**

Use resource hints to preconnect/preload domains/assets that the page will need, such as 3rd party JS.

Include SVG and PNG favicons. I recommend an SVG and PNGs at 192px and 180px square. [SVGs should support dark mode](https://stackoverflow.com/a/67190894).

### Templates

Store page template code in an /includes file, not in the page_templates/ folder.

Create a blog category for the main blog URL, instead of a page template. This will make styling archives easier and more consistent.

Make sure that category, tag, date, and author archives, as well as search results, are either styled or disabled.

### CSS

Use SASS (SCSS syntax).

Attempt to limit color palette and margin variations by using variables.

Use CSS grid and flexbox liberally, since they're fully supported in all modern browsers. Avoid using floats for layout.

Avoid using display: none for content that should be visible to screen readers.

Avoid transitions/animations on attributes other than opacity and transform when possible. 

Consolidate CSS into one stylesheet for global styles (layout, typography, header/footer, etc) and separate stylesheets for each page template (home, about, contact, etc).

### JS

Manage JS using webpack. Use dynamic imports for large dependencies. 

Don't use jQuery. Be mindful of excessive dependencies for user-facing JS.

Load JS in the footer, with defer and/or async attributes when possible.

Review the JS files being loaded on the frontend and dequeue any that aren't needed. The WP core, parent themes, and plugins all tend load JS that can be removed.

## Images & Icons

Save icons in the theme files as SVGs. For sites with a small number of icons on the page, include them inline. For sites with hundreds of icons shown at once, use a sprite. 

Make sure that SVGs don't have image data encoded in them. Most SVG icons are smaller than 2kb.

Optimize all image assets saved to the theme before committing the files, using a tool like [ImageOptim](https://imageoptim.com/). Images don't need to be optimized as part of the build process, since this is one-time operation.

### Fonts

Serve fonts from the WP/CDN server whenever possible, instead of using a 3rd party CDN like Adobe or Typography.com. Serving 1st party fonts reduces DNS lookup time and lets the theme fine-tune the font loading process by, for example:

- Loading the font at the top of <head>
- Adding font preload hints
- Managing font variations

[google-webfonts-helper](https://google-webfonts-helper.herokuapp.com/fonts) is a useful app for extracting the core font files from Google Fonts. .woff and .woff2 formats are sufficient for all browsers in 2021.

An ideal font stack is `'system-ui', -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif`, which tells the browser to use the most modern sans serif font available in each OS. This will look good and perform excellently on all systems, but of course, may not be doable design-wise.

## Live Site Improvement Recommendations

For a project that's already live, implementing all of the above best practices may not be feasible. Here's a list of "low hanging fruit" recommended improvements, in order of importance:

- Migrate the site to WPEngine and put it behind GES or Cloudflare
- Make sure the site is fully SSL'd
- Isolate the site's custom code files into a theme, create a repo for it, and move to git deployment
- Add Postmark for email fulfillment
- Install these plugins, which are all low-configuration and shouldn't interfere with existing functionality:
	- Forbid Pwned Passwords
	- Disable REST API
	- Disable Emojis
	- Limit Login Attempts Reloaded
	- wp-password-bcrypt
	- Disable Comments
	- Disable Embeds
	- Disable Emojis
	- Imsanity
