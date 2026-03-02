# Marketing Page Setup (PowerAdmin) via API/CLI

This documents how to seed content and wire a Marketing home page for the PowerAdmin site without the demo starter.

## Prereqs
- dotCMS clean install (no starter), PowerAdmin site created, live, and set as default; hostnames: `developer.black` (+ optional `app.developer.black`).
- Admin creds: `admin@dotcms.com / SecureP@ssw0rd!` (change after setup).
- dotcli configured to `https://app.developer.black` (profile `prod` active).

## Seed content via REST
Use Basic auth or a token. Examples below use basic auth. Run from any shell on the host with asset files available.

### Hero banners (3)
```bash
cd dotcms-dev/temp_theme_extract/PowerAdmin-pro
for i in 1 2 3; do
  curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST \
    -F stName=HeroBanner -F contentHost=PowerAdmin \
    -F title="Modern Cloud Ops $i" \
    -F subtitle="Security, observability, automation" \
    -F buttonText="Get Started" -F buttonUrl="/" \
    -F image=@assets/img/slides/slide-hero-$i.webp \
    https://app.developer.black/api/content/save
  echo
done
```

### Feature items (5)
```bash
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"FeatureItem","contentHost":"PowerAdmin","title":"Real-time Chat","iconClass":"bi bi-chat-dots","description":"Secure team chat with message threading and reactions.","linkUrl":"/apps-chat"}' https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"FeatureItem","contentHost":"PowerAdmin","title":"Unified Inbox","iconClass":"bi bi-envelope","description":"Email, tickets, and notifications in one queue.","linkUrl":"/apps-email"}' https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"FeatureItem","contentHost":"PowerAdmin","title":"Calendar & Tasks","iconClass":"bi bi-calendar-event","description":"Schedule events and track tasks across teams.","linkUrl":"/apps-calendar"}' https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"FeatureItem","contentHost":"PowerAdmin","title":"Kanban Boards","iconClass":"bi bi-kanban","description":"Visual pipelines with swimlanes and assignees.","linkUrl":"/apps-kanban"}' https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"FeatureItem","contentHost":"PowerAdmin","title":"File Manager","iconClass":"bi bi-folder","description":"Versioned documents with permissions and preview.","linkUrl":"/apps-file-manager"}' https://app.developer.black/api/content/save
```

### Team members (3)
```bash
cd dotcms-dev/temp_theme_extract/PowerAdmin-pro
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST \
  -F stName=TeamMember -F contentHost=PowerAdmin \
  -F fullName="Alex Thompson" -F role="CTO" \
  -F bio="Leads platform reliability and automation." \
  -F socialLinkedin="https://linkedin.com" \
  -F photo=@assets/img/avatars/avatar-4.webp \
  https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST \
  -F stName=TeamMember -F contentHost=PowerAdmin \
  -F fullName="Priya Desai" -F role="Head of Product" \
  -F bio="Owns roadmap, research, and UX delivery." \
  -F socialTwitter="https://twitter.com" \
  -F photo=@assets/img/avatars/avatar-5.webp \
  https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST \
  -F stName=TeamMember -F contentHost=PowerAdmin \
  -F fullName="Sam Rivera" -F role="Customer Success" \
  -F bio="Drives adoption and onboarding for enterprise teams." \
  -F socialLinkedin="https://linkedin.com" \
  -F photo=@assets/img/avatars/avatar-7.webp \
  https://app.developer.black/api/content/save
```

### Testimonials (2)
```bash
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"Testimonial","contentHost":"PowerAdmin","quote":"PowerAdmin centralizes our ops and security dashboards.","authorName":"Jordan Lee","authorRole":"VP Engineering, Northwind"}' https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"Testimonial","contentHost":"PowerAdmin","quote":"We cut incident response time by 40% after rolling this out.","authorName":"Taylor Morgan","authorRole":"Head of SRE, Acme Labs"}' https://app.developer.black/api/content/save
```

### Call To Action + Intro
```bash
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"CallToAction","contentHost":"PowerAdmin","headline":"Ready to modernize ops?","subtext":"Deploy dashboards, workflows, and alerts in one place.","buttonText":"Book a demo","buttonUrl":"/contact"}' https://app.developer.black/api/content/save
curl -s -u 'admin@dotcms.com:SecureP@ssw0rd!' -X POST -H "Content-Type: application/json" -d '{"stName":"webPageContent","contentHost":"PowerAdmin","title":"Dashboard Intro","body":"<p>Monitor usage, revenue, and support from one admin workspace. Customize widgets, alerts, and shortcuts for your team.</p>"}' https://app.developer.black/api/content/save
```

## Template and containers (UI quickest)
Create six containers: hero, features, team, testimonials, cta, intro. Then create a Template "Marketing" with those containers laid out (hero full width; features grid; team grid; testimonials; CTA full-width; intro text).

Assign the template to a home Page under PowerAdmin (e.g., `/index`), place the containers, and bind content pulls to the above content types. Publish the page. PowerAdmin is already default, so `https://developer.black` will serve it.

## Notes
- Assets already pushed to PowerAdmin: `/images/hero/slide-hero-{1,2,3}.webp`, `/images/team/avatar-{1,2,3}.webp`.
- Custom content types exist on the server (HeroBanner, FeatureItem, TeamMember, Testimonial, CallToAction, webPageContent).
- To avoid demo push errors, keep demo content-type JSONs removed.
