

    const PREF_FIELDS = [
      { key: "text", label: "Предпочтение", type: "textarea", placeholder: "Например: любит кофе без сахара", required: true },
      { key: "level", label: "Важность", type: "level" }
    ];

    function renderPrefs(app, key) {
      const s = SECTIONS[key];
      app.append(h("h2", {}, s.emoji + " " + s.title));

      const legend = h("div", { class: "legend" });
      Object.keys(LEVELS).forEach(k => {
        legend.append(h("div", {}, h("div", { class: "dot " + k }), LEVELS[k].name + " — " + LEVELS[k].hint));
      });
      app.append(legend);

      const chips = h("div", { class: "chips" });
      [["all", "Все"], ["red", "🔴 Важно"], ["yellow", "🟡 Желательно"], ["green", "🟢 Можно без"]].forEach(([val, name]) => {
        chips.append(h("button", {
          class: "chip" + (prefFilter[key] === val ? " active" : ""),
          onclick: () => { prefFilter[key] = val; render(); }
        }, name));
      });
      app.append(chips);

      let list = state.prefs[key].slice();
      if (prefFilter[key] !== "all") list = list.filter(p => p.level === prefFilter[key]);
      list.sort((a, b) => LEVEL_ORDER[a.level] - LEVEL_ORDER[b.level]);

      if (!list.length) app.append(h("div", { class: "empty" }, "Здесь пока ничего нет"));

      list.forEach(p => {
        app.append(h("div", { class: "card", onclick: () => {
          openForm({
            title: "Изменить предпочтение", fields: PREF_FIELDS, values: p,
            onSave: v => { p.text = v.text; p.level = v.level; save(); },
            onDelete: () => { state.prefs[key] = state.prefs[key].filter(x => x.id !== p.id); save(); }
          });
        } },
          h("div", { class: "dot " + p.level }),
          h("div", { class: "body" }, h("div", { class: "name", style: "font-weight:500;white-space:pre-wrap;word-break:break-word" }, p.text))));
      });

      addFab("+ Добавить предпочтение", () => {
        openForm({
          title: "Новое предпочтение", fields: PREF_FIELDS,
          onSave: v => { state.prefs[key].push({ id: uid(), text: v.text, level: v.level }); save(); }
        });
      });
    }

    render();
  </script>
</body>
</html>
