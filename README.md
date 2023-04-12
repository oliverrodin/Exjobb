# Exjobb


## Komponentlogik utan React Query
```tsx
const HomePage = () => {
  const { t } = useTranslation(['news', 'common']);
  const theme = useTheme();
  const { loggedInUser } = useGlobals();
  const { categoriesOptions } = useFetchCategories(true);
  const isMounted = useIsMounted();
  // Local states
  const [optionsName, setOptionsName] = useState<string>('Alla');
  const [loading, setLoading] = useState(false);
  const [data, setData] = useState<Array<NewsResponse> | undefined>([]);
  const [search, setSearch] = useState('');
  const [pageIndex, setPageIndex] = useState(0);
  const [totalPages, setTotalPages] = useState<number>();
  const [hasMore, setHasMore] = useState<boolean>(false);
  const [activeNewsItemId, setActiveNewsItemId] = useState<string>();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));

  // Sätter title i tab
  useChangeTitle(t('common:pageTabs.news'));

  const debouncedSearchTerm = useDebounce(search, 300);

  const observer: any = useRef();
  const lastNewsRef = useCallback(
    (node: any) => {
      if (loading) return;
      if (observer.current) observer.current.disconnect();
      observer.current = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting && hasMore) {
          setPageIndex((prev) => prev + 1);
        }
      });
      if (node) observer.current.observe(node);
    },
    [loading, hasMore],
  );

  // Hämta nyheter
  const fetchNews = useCallback(async () => {
    const newsApi = new NewsAPI();
    setLoading(true);
    ConsoleHelper.log('fetching news on homePage.js');
    try {
      const reqOptions = new NewsOptions();
      reqOptions.Category = optionsName ?? 'Alla';
      reqOptions.PageIndex = pageIndex;
      reqOptions.PageSize = 10;
      reqOptions.Filter = debouncedSearchTerm;

      const res = await newsApi.GetAll(reqOptions);
      if (isMounted()) {
        ConsoleHelper.log(
          'FetchNews result for pageIndex',
          pageIndex,
          res.Results,
        );
        if (res.Results && res.Results.length > 0) {
          if (pageIndex === 0) {
            setData(res.Results);
            setTotalPages(res.TotalPages);
          } else {
            setData((prev) => [...(prev || []), ...res.Results]);
            setTotalPages(res.TotalPages);
          }
          setHasMore(res.Results.length >= 10);
        } else {
          setData(undefined);
        }
      }
    } catch (error: any) {
      if (isMounted()) {
        ConsoleHelper.log(error);
        setData(undefined);
      }
    } finally {
      if (isMounted()) {
        setLoading(false);
      }
    }
  }, [optionsName, pageIndex, debouncedSearchTerm, isMounted]);

  // Hämta om news by id och ersätt motsvarande guid/id i befitnlig array efter action för att få nytt data utan at fucka upp paging
  const replaceNewsItemAfterAction = async (item: NewsResponse) => {
    const newsApi = new NewsAPI();
    try {
      const updatedItem = await newsApi.Get(item?.GUID);
      // data?.map((obj) => (updatedItem.ID === obj.ID ? updatedItem : obj));
      const updatedArray = data?.map((obj) =>
        updatedItem.GUID === obj.GUID ? updatedItem : obj,
      );
      setData(updatedArray); // Funkade inte utan setData?
      ConsoleHelper.log('replaceNewsItemAfterAction färdig');
    } catch (err) {
      ConsoleHelper.log('ERROR replace nyhets data efter action', err);
    }
  };

  useEffect(() => {
    fetchNews();
  }, [fetchNews]);

  const handleSearch = (event: any) => {
    setPageIndex(0);
    setSearch(event.target.value);
  };

  const handleActiveNewsItemId = (newsItemId: string) => {
    setActiveNewsItemId(
      activeNewsItemId === newsItemId ? undefined : newsItemId,
    );
  };

```

## Komponentlogik med React Query
```tsx
const HomePage = () => {
  const { t } = useTranslation(['news', 'common']);
  const theme = useTheme();

  // Global states
  const { loggedInUser } = useGlobals();

  // Local states
  const [optionsName, setOptionsName] = useState<string>('Alla');
  const [search, setSearch] = useState('');
  const [activeNewsItemId, setActiveNewsItemId] = useState<string>();
  const isMobile = useMediaQuery(theme.breakpoints.down('md'));

  // Sätter title i tab
  useChangeTitle(t('common:pageTabs.news'));

  const debouncedSearchTerm = useDebounce(search, 300);

  // React Query
  const categoriesOptions = useFetchNewsCategories(true);
  const { fetchNextPage, hasNextPage, isFetchingNextPage, data, isLoading } =
    useFetchNews({
      Category: optionsName ?? 'Alla',
      Filter: debouncedSearchTerm,
    });

  const intObserver: any = useRef();
  const lastNewsRef = useCallback(
    (node) => {
      if (isFetchingNextPage) return;

      if (intObserver.current) intObserver.current.disconnect();

      intObserver.current = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting && hasNextPage) {
          ConsoleHelper.log('We are near the last post!');
          fetchNextPage();
        }
      });

      if (node) intObserver.current.observe(node);
    },
    [isFetchingNextPage, hasNextPage, fetchNextPage],
  );

  const handleSearch = (event: any) => {
    setSearch(event.target.value);
  };

  const handleActiveNewsItemId = (newsItemId: string) => {
    setActiveNewsItemId(
      activeNewsItemId === newsItemId ? undefined : newsItemId,
    );
  };
  console.log(data?.pages);
  return (
```
