<h1> DragEvent
    
</h1>

<p> Control UI (e.g. View control) with receive drag event : </p>

`````java
new View.OnDragListener(){
            @Override
            public boolean onDrag(View v, DragEvent event) {
                //<code> onDrag() </code> called whenever dragEvent occur.
                //only the views can receive event continually, that returned <code> true </code> at <code> event.getAction==DragEvent.ACTION_DRAG_STARTED </code> case in <code> onDrag() </code> block.
                if (!(v instanceof MaterialCardView)) {
                    return false;
                }

                FrameLayout parentView = (FrameLayout) v.getParent();
                RecyclerView cardRecyclerView = (RecyclerView) parentView.getParent();
                int position = cardRecyclerView.getChildAdapterPosition(parentView);
                CardAdapter cardAdapter = (CardAdapter) cardRecyclerView.getAdapter();
                ItemCardFrameBinding cardFrameBinding = ((ContactCardViewHolder) cardRecyclerView.getChildViewHolder(parentView)).getBinding();
                CardDTO cardDTO = cardFrameBinding.getCard();
                int targetSeqNo = cardDTO.getSeqNo();
                int targetContainerNo = cardDTO.getContainerNo();
                int targetBossNo = cardDTO.getBossNo();
                List<Pair<CardDTO, CardState>> targetContainerPresentCardItems = mPresentData.get(targetContainerNo);

                switch (event.getAction()) {
                    case DragEvent.ACTION_DRAG_ENTERED:
                        if (!targetContainerPresentCardItems.get(targetSeqNo).second.isPersistence()){
                            ((MaterialCardView) v).setCardBackgroundColor(Color.YELLOW);
                            break;
                        }

                        //Create CardDTO
                        CardDTO newCardDTO = new CardDTO.Builder().bossNo(targetBossNo).seqNo(targetSeqNo + 1).containerNo(targetContainerNo).build();
                        if (targetContainerPresentCardItems.size() > targetSeqNo + 1) {
                            toPushBackSeqListItems(targetContainerPresentCardItems, targetSeqNo + 1);
                        }
                        targetContainerPresentCardItems.add(targetSeqNo + 1, Pair.create(newCardDTO, new CardState(CardState.FRONT_DISPLAYING, false)));
                        Optional.ofNullable(cardAdapter).ifPresent((cardAdapter1 -> cardAdapter.notifyItemInserted(targetSeqNo + 1)));
                        cardRecyclerView.smoothScrollToPosition(targetSeqNo + 1);
                        break;
                    case DragEvent.ACTION_DRAG_EXITED:
                        if (targetContainerPresentCardItems.get(targetSeqNo).second.isPersistence()){
                            break;
                        }
                        targetContainerPresentCardItems.remove(targetSeqNo);
                        if (targetContainerPresentCardItems.size() > targetSeqNo) {
                            takeListItemsSeqBack(targetContainerPresentCardItems, targetSeqNo);
                        }
                        Optional.ofNullable(cardAdapter).ifPresent((cardAdapter1 -> cardAdapter.notifyItemRemoved(targetSeqNo)));
                        break;
                    case DragEvent.ACTION_DRAG_ENDED:
//                    ((MaterialCardView) v).setCardBackgroundColor(v.getResources().getColor(R.color.colorPrimary));
                        break;
                    case DragEvent.ACTION_DROP:
                        if (targetContainerPresentCardItems.get(targetSeqNo).second.isPersistence()){
                            break;
                        }
                        targetContainerPresentCardItems.get(targetSeqNo).second.toPersistence();
//                    FrameLayout parentView = (FrameLayout) v.getParent();
//                    RecyclerView cardRecyclerView = (RecyclerView) parentView.getParent();
//                    int position = cardRecyclerView.getChildAdapterPosition(parentView);
//                    CardAdapter cardAdapter= (CardAdapter)cardRecyclerView.getAdapter();
//                    ItemCardFrameBinding cardFrameBinding = ((ContactCardViewHolder)cardRecyclerView.getChildViewHolder(parentView)).getBinding();
//                    CardDTO cardDTO = cardFrameBinding.getCard();
//                    int targetCardNo = cardDTO.getCardNo();
//                    int targetSeqNo = cardDTO.getSeqNo();
//                    int targetContainerNo = cardDTO.getContainerNo();
//                    int targetBossNo = cardDTO.getBossNo();
//                    //Create CardDTO
//                    CardDTO newCardDTO = new CardDTO.Builder().bossNo(targetBossNo).seqNo(targetSeqNo).containerNo(targetContainerNo).build();
//                    //align seqNo
//
//                    //add CardDTO to present List
//
//                    //sort??
//
//                    //insert CardDTO data
//                    mCardRepository.insertCard(newCardDTO.toEntity());
                        break;
                }
                return true;
            }
        };
`````

