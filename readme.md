## have better solution

when caliing the api we can use the TPL library in a ay that instead of using foreach use task parrallel and add into the array so that call is much quiker like
<p>
CODE[
public IEnumerable<Planet> GetAllPlanets()
{
    var allPlanetsWithTheirMoons = new Collection<Planet>();

    var response = _httpClientService.Client
        .GetAsync(UriPath.GetAllPlanetsWithMoonsQueryParameters)
        .Result;

    //If the status code isn't 200-299, then the function returns an empty collection.
    if (!response.IsSuccessStatusCode)
    {
        Logger.Instance.Warn($"{LoggerMessage.GetRequestFailed}{response.StatusCode}");
        return allPlanetsWithTheirMoons;
    }

    var content = response.Content.ReadAsStringAsync().Result;

    //The JSON converter uses DTO's, that can be found in the DataTransferObjects folder, to deserialize the response content.
    var results = JsonConvert.DeserializeObject<JsonResult<PlanetDto>>(content);

    
    if (results == null) return allPlanetsWithTheirMoons;

    Parallel.ForEach(results.Bodies, planet =>
    {
        if (planet.Moons != null)
        {
            var newMoonsCollection = new Collection<MoonDto>();
            Parallel.ForEach(planet.Moons, moon =>
            {
                var moonResponse = _httpClientService.Client
                    .GetAsync(UriPath.GetMoonByIdQueryParameters + moon.URLId)
                    .Result;
                var moonContent = moonResponse.Content.ReadAsStringAsync().Result;
                newMoonsCollection.Add(JsonConvert.DeserializeObject<MoonDto>(moonContent));
            });

            planet.Moons = newMoonsCollection;
        }

        allPlanetsWithTheirMoons.Add(new Planet(planet));
    });

    return allPlanetsWithTheirMoons;
}
]</p>