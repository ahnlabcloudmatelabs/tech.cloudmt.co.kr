---
title: Tags
description: Filter posts by tags
---
{{ $tags := $.Site.Taxonomies.tags.ByCount }}
{{ $v1 := where $tags "Count" ">=" 3 }}
{{ $v2 := where $v1 "Term" "not in" (slice "hugo" "tags" "rss") }}
{{ range $v2 }}
{{ if .Term }}
{{ $tagURL := printf "tags/%s" .Term | relURL }}
<li>
<a class="tag-button" href="{{ $tagURL }}">
    {{ .Term }}<sup>{{ .Count }}</sup>
</a>
</li>
{{ end }}
{{ end }}