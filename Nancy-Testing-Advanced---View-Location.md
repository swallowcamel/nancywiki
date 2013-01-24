# Nancy Testing Advanced - View Location

By default Nancy unit tests will struggle to find view files when you run your tests.

You can create the following file in your test project to help Nancy out.

`
public class TestingRootPathProvider : IRootPathProvider
{
    private static readonly string RootPath;

    static TestingRootPathProvider()
    {
        var directoryName = Path.GetDirectoryName(typeof (Bootstrapper).Assembly.CodeBase);

        if (directoryName != null)
        {
            var assemblyPath = directoryName.Replace(@"file:\", string.Empty);

            RootPath = Path.Combine(assemblyPath, "..", "..", "..", "Escape.Web");
        }
    }

    public string GetRootPath()
    {
        return RootPath;
    }
}
`