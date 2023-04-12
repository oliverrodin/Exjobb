# Exjobb


## Komponentlogik utan React Query
```tsx
export const ChatActionDropdown = (props: ChatActionDropdownProps) => {
  const {
    refetchChatRoom,
    chatRoom,
    members,
    onClickLeave,
    onClickAddMember,
    chatRoomOpen,
    onClickJoinChat,
  } = props;
  const { t } = useTranslation(['chat']);

  // Global States
  const { loggedInUser } = useGlobals();
  const isMounted = useIsMounted();

  // Local state
  const [anchorEl, setAnchorEl] = React.useState<null | HTMLElement>(null);
  const [loading, setLoading] = useState<boolean>(false);
  const [newName, setNewName] = useState<string>(chatRoom.Name);
  const [isPublic, setIsPublic] = useState<boolean>(false);
  const [isPinned, setIsPinned] = useState<boolean>(false);
  const [showNameDialog, setShowNameDialog] = useState<boolean>(false);
  const [openDeleteChatDialog, setOpenDeleteChatDialog] =
    useState<boolean>(false);

  const userIsSystemAdmin = loggedInUser?.UserType === 1;
  const userIsChatRoomAdmin = loggedInUser?.GUID === chatRoom?.UserID;
  const loggedInChatMember = members?.find(
    (m) => m.UserID === loggedInUser?.GUID,
  );

  // Handle anchor and open/close menu
  const open = Boolean(anchorEl);
  const handleClick = (event: React.MouseEvent<HTMLElement>) => {
    setAnchorEl(event.currentTarget);
  };
  const handleClose = () => {
    setAnchorEl(null);
  };

  const onSaveChatRoomName = async () => {
    const api = new ChatRoomAPI();
    setLoading(true);
    const currentUserIDs = members?.map((m) => m.UserID);
    const chatRoomData = {
      ID: chatRoom.ID,
      Type: chatRoom.Type,
      UserIDs: currentUserIDs,
      Name: newName,
    };

    try {
      await api.Update(chatRoomData);
      if (isMounted()) {
        refetchChatRoom();
      }
    } catch (err: any) {
      if (isMounted()) {
        toast.error(
          `${t('somethingWentWrong')} 
          ${
            err?.response?.data?.Message !== 'An error has occurred.'
              ? err?.response?.data?.Message
              : t('takeABreathAndTryAgain')
          }`,
        );
      }
    } finally {
      if (isMounted()) {
        setLoading(false);
      }
    }
  };

  const onPressTogglePrivate = async (value: boolean) => {
    const api = new ChatRoomAPI();
    setIsPublic(value);
    setLoading(true);
    const currentUserIDs = members?.map((m) => m.UserID);
    const chatRoomData = {
      ID: chatRoom.ID,
      Name: chatRoom.Name,
      Type: value ? 1 : 2, // Om toggle isPublic = true sätt Type 1, om isPublic = false sätt Type 2
      UserIDs: currentUserIDs,
    };

    try {
      await api.Update(chatRoomData);
      if (isMounted()) {
        refetchChatRoom();
      }
    } catch (err: any) {
      if (isMounted()) {
        ConsoleHelper.log('ERR toggle type', err);
        setIsPublic(chatRoom.Type === 1);

        toast.error(
          `${t('somethingWentWrong')}
          ${
            err?.response?.data?.Message !== 'An error has occurred.'
              ? err?.response?.data?.Message
              : t('takeABreathAndTryAgain')
          }`,
        );
      }
    } finally {
      if (isMounted()) {
        setLoading(false);
      }
    }
  };

  const onPressPinRoom = async (value: boolean) => {
    setLoading(true);
    setIsPinned(value);
    const api = new ChatRoomAPI();
    try {
      const res = await api.TogglePinRoom(chatRoom.ID);
      if (isMounted()) {
        ConsoleHelper.log('RES toggle pin', res);
        refetchChatRoom();
      }
    } catch (err: any) {
      if (isMounted()) {
        ConsoleHelper.log('ERROR pin room', err);
        toast.error(
          `${t('somethingWentWrong')} 
          ${
            err?.response?.data?.Message !== 'An error has occurred.'
              ? err?.response?.data?.Message
              : t('takeABreathAndTryAgain')
          }`,
        );
        setIsPinned(chatRoom.IsPinned);
      }
    } finally {
      if (isMounted()) {
        setLoading(false);
      }
    }
  };

  const deleteChatRoom = async (ID: number) => {
    setLoading(true);
    const api = new ChatRoomAPI();
    try {
      await api.Delete(ID);
      if (isMounted()) {
        toast.success(t('Done'));
      }
    } catch (err: any) {
      if (isMounted()) {
        toast.error(t('somethingWentWrong'), t('takeABreathAndTryAgain'));
      }
    } finally {
      if (isMounted()) {
        setLoading(false);
      }
    }
  };

  useEffect(() => {
    setIsPublic(chatRoom.Type === 1);
    setIsPinned(chatRoom.IsPinned);
  }, [chatRoom]);

  return (
		// code...
  )
```

## Komponentlogik med React Query
```tsx
export const ChatActionDropdown = (props: ChatActionDropdownProps) => {
  const { onClickLeave, onClickAddMember, chatRoomOpen, onClickJoinChat } =
    props;
  const { t } = useTranslation(['chat']);

  // Global States
  const { loggedInUser, currentChatRoom } = useGlobals();

  // useQuery
  const members = useFetchChatRoomEmployees({
    ChatRoomID: currentChatRoom?.ID,
  });
  const chatRoom = useFetchChatRoomById(currentChatRoom?.ID ?? 0);

  // useMutation
  const updateChatRoomName = useMutationUpdateChatRoomName(
    chatRoom?.data?.ID ?? 0,
  );
  const togglePrivate = useMutationTogglePrivate(chatRoom?.data?.ID ?? 0);
  const togglePinRoom = useMutationTogglePinRoom({
    chatRoomId: chatRoom?.data?.ID ?? 0,
    isPinned: chatRoom.data?.IsPinned ?? false,
  });
  const deleteChatRoom = useMutationDeleteChatRoom();

  // Local state
  const [anchorEl, setAnchorEl] = React.useState<null | HTMLElement>(null);
  const [newName, setNewName] = useState<string>(chatRoom?.data?.Name ?? '');
  const [showNameDialog, setShowNameDialog] = useState<boolean>(false);
  const [openDeleteChatDialog, setOpenDeleteChatDialog] =
    useState<boolean>(false);

  const userIsSystemAdmin = loggedInUser?.UserType === 1;
  const userIsChatRoomAdmin = loggedInUser?.GUID === chatRoom.data?.UserID;
  const loggedInChatMember = members.data?.find(
    (m) => m.UserID === loggedInUser?.GUID,
  );

  // Handle anchor and open/close menu
  const open = Boolean(anchorEl);
  const handleClick = (event: React.MouseEvent<HTMLElement>) => {
    setAnchorEl(event.currentTarget);
  };
  const handleClose = () => {
    setAnchorEl(null);
  };

  const onSaveChatRoomName = async () => {
    let currentUserIDs = [] as string[];
    if (members.data) {
      currentUserIDs = members.data.map((m) => m.UserID);
    }
    if (chatRoom.data) {
      const chatRoomData = {
        ID: chatRoom.data.ID,
        Type: chatRoom.data.Type,
        UserIDs: currentUserIDs,
        Name: newName,
      };
      updateChatRoomName.mutate(chatRoomData, {
        onError: () => {
          toast.error(`${t('somethingWentWrong')}`);
        },
      });
    }
  };

  const onPressTogglePrivate = async (value: boolean) => {
    let currentUserIDs = [] as string[];
    if (members.data) {
      currentUserIDs = members.data.map((m) => m.UserID);
    }
    if (chatRoom.data) {
      const chatRoomData = {
        ID: chatRoom.data.ID,
        Name: chatRoom.data.Name,
        Type: value ? 1 : 2, // Om toggle isPublic = true sätt Type 1, om isPublic = false sätt Type 2
        UserIDs: currentUserIDs,
      };
      togglePrivate.mutate(chatRoomData, {
        onError: () => {
          toast.error(`${t('somethingWentWrong')}`);
        },
      });
    }
  };

  const onPressPinRoom = async (value: boolean) => {
    if (chatRoom.data) {
      togglePinRoom.mutate(chatRoom.data.ID, {
        onError: () => {
          toast.error(`${t('somethingWentWrong')}`);
        },
      });
    }
  };

  const deleteChatRoomHandler = async (ID: number) => {
    deleteChatRoom.mutate(ID, {
      onSuccess: () => {
        setOpenDeleteChatDialog(false);
        chatRoomOpen(false);
        toast.success(t('Done'));
      },
      onError: () => {
        toast.error(t('somethingWentWrong'), t('takeABreathAndTryAgain'));
      },
    });
  };

  return (
    // code...
  )
```
