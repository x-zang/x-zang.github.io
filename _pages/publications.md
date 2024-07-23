---
layout: archive
title: "Publications"
permalink: /publications/
author_profile: true
---

{% if author.googlescholar %}
  You can also find my articles on <u><a href="{{https://scholar.google.com/citations?user=tZhzac0AAAAJ}}">my Google Scholar profile</a>.</u>
{% endif %}

{% include base_path %}

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}

- **Zang, X.C.**, Li, X., Metcalfe, K., Ben-Yehezkel, T., Kelley, R., and Shao, M.  
“Anchorage accurately assembles anchor-flanked synthetic long reads”.  
In Proc. 24th Int’l Workshop on Algorithms in Bioinformatics (**WABI’24**),  
[software](https://github.com/Shao-Group/anchorage)

- Zahin, T., Shi, Q., **Zang, X.C.**, and Shao, M.  
“Accurate assembly of circular RNAs with TERRACE”.  
In Proc. 28th Int’l Conf. Comput. Mol. Biol. (**RECOMB’24**),  
[proceedings](https://link.springer.com/chapter/10.1007/978-1-0716-3989-4_49),
[preprint](https://www.biorxiv.org/content/10.1101/2024.02.09.579380v1),
[software](https://github.com/Shao-Group/TERRACE)
