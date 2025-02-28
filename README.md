var groupedData = viewWeeklyKpiDataList
    .GroupBy(x => new { x.ShopCode, x.AreaName })
    .Select(areaGroup => new Area
    {
        DisplayName = areaGroup.Key.AreaName,
        Kpis = areaGroup
            .GroupBy(k => k.KPIID)
            .Select(kpiGroup => new Kpi
            {
                KpiId = int.TryParse(kpiGroup.First().KPIID, out var kpiId) ? kpiId : null,
                Name = kpiGroup.First().KpiName,
                ParentId = kpiGroup.First().ParentKPIID,
                Values = new List<KpiValue>
                {
                    new KpiValue { Title = "Wk52", Value = kpiGroup.Sum(k => double.TryParse(k.Wk52, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk1", Value = kpiGroup.Sum(k => double.TryParse(k.Wk1, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk2", Value = kpiGroup.Sum(k => double.TryParse(k.Wk2, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk3", Value = kpiGroup.Sum(k => double.TryParse(k.Wk3, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk4", Value = kpiGroup.Sum(k => double.TryParse(k.Wk4, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk5", Value = kpiGroup.Sum(k => double.TryParse(k.Wk5, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk6", Value = kpiGroup.Sum(k => double.TryParse(k.Wk6, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk7", Value = kpiGroup.Sum(k => double.TryParse(k.Wk7, out var v) ? v : 0) },
                    new KpiValue { Title = "Wk8", Value = kpiGroup.Sum(k => double.TryParse(k.Wk8, out var v) ? v : 0) },
                    new KpiValue { Title = "WeekTarget", Value = kpiGroup.Sum(k => double.TryParse(k.WeekTarget, out var v) ? v : 0) },
                    new KpiValue { Title = "YearEndTarget", Value = kpiGroup.Sum(k => double.TryParse(k.YearEndTarget, out var v) ? v : 0) }
                }
            })
            .ToList()
    })
    .ToList();

// 🔹 Build the Parent-Child KPI Relationship
foreach (var area in groupedData)
{
    var kpiLookup = area.Kpis.ToDictionary(k => k.KpiId);
    foreach (var kpi in area.Kpis)
    {
        if (kpi.ParentId.HasValue && kpiLookup.ContainsKey(kpi.ParentId.Value))
        {
            kpiLookup[kpi.ParentId.Value].Children.Add(kpi);
        }
    }
    area.Kpis = area.Kpis.Where(k => !k.ParentId.HasValue).ToList(); // Keep only root KPIs
}

// 🔹 Generate Final Response
var response = new WeeklyKpiResponse
{
    ShopCode = viewWeeklyKpiDataList.First().ShopCode,
    Areas = groupedData
};
