---
title: 源码分析(八) ListView和RecyclerView机制
date: 2020-03-09 22:09:13
tags:
 - 源码分析
---
ListView和RecyclerView机制分析
<!--more-->

RecyclerBin机制
这个机制其实就是ListView的缓存复用机制，可以让我们在加载非常多的item时也不会OOM，这个机制主要是通过AbsListView中到的RecyclerBin类完成的，意思也就是说所有的AbsListView的子类都实现了这个机制，他的源码如下(部分内容作了一些删减)
```
/**
 * The RecycleBin facilitates reuse of views across layouts. The RecycleBin
 * has two levels of storage: ActiveViews and ScrapViews. ActiveViews are
 * those views which were onscreen at the start of a layout. By
 * construction, they are displaying current information. At the end of
 * layout, all views in ActiveViews are demoted to ScrapViews. ScrapViews
 * are old views that could potentially be used by the adapter to avoid
 * allocating views unnecessarily.
 * 
 * @see android.widget.AbsListView#setRecyclerListener(android.widget.AbsListView.RecyclerListener)
 * @see android.widget.AbsListView.RecyclerListener
 */
class RecycleBin {
	private RecyclerListener mRecyclerListener;
 
	/**
	 * The position of the first view stored in mActiveViews.
	 */
	private int mFirstActivePosition;
 
	/**
	 * Views that were on screen at the start of layout. This array is
	 * populated at the start of layout, and at the end of layout all view
	 * in mActiveViews are moved to mScrapViews. Views in mActiveViews
	 * represent a contiguous range of Views, with position of the first
	 * view store in mFirstActivePosition.
	 */
	private View[] mActiveViews = new View[0];
 
	/**
	 * Unsorted views that can be used by the adapter as a convert view.
	 */
	private ArrayList<View>[] mScrapViews;
 
	private int mViewTypeCount;
 
	private ArrayList<View> mCurrentScrap;
 
	/**
	 * Fill ActiveViews with all of the children of the AbsListView.
	 * 
	 * @param childCount
	 *            The minimum number of views mActiveViews should hold
	 * @param firstActivePosition
	 *            The position of the first view that will be stored in
	 *            mActiveViews
	 */
	void fillActiveViews(int childCount, int firstActivePosition) {
		if (mActiveViews.length < childCount) {
			mActiveViews = new View[childCount];
		}
		mFirstActivePosition = firstActivePosition;
		final View[] activeViews = mActiveViews;
		for (int i = 0; i < childCount; i++) {
			View child = getChildAt(i);
			AbsListView.LayoutParams lp = (AbsListView.LayoutParams) child.getLayoutParams();
			// Don't put header or footer views into the scrap heap
			if (lp != null && lp.viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				// Note: We do place AdapterView.ITEM_VIEW_TYPE_IGNORE in
				// active views.
				// However, we will NOT place them into scrap views.
				activeViews[i] = child;
			}
		}
	}
 
	/**
	 * Get the view corresponding to the specified position. The view will
	 * be removed from mActiveViews if it is found.
	 * 
	 * @param position
	 *            The position to look up in mActiveViews
	 * @return The view if it is found, null otherwise
	 */
	View getActiveView(int position) {
		int index = position - mFirstActivePosition;
		final View[] activeViews = mActiveViews;
		if (index >= 0 && index < activeViews.length) {
			final View match = activeViews[index];
			activeViews[index] = null;
			return match;
		}
		return null;
	}
 
	/**
	 * Put a view into the ScapViews list. These views are unordered.
	 * 
	 * @param scrap
	 *            The view to add
	 */
	void addScrapView(View scrap) {
		AbsListView.LayoutParams lp = (AbsListView.LayoutParams) scrap.getLayoutParams();
		if (lp == null) {
			return;
		}
		// Don't put header or footer views or views that should be ignored
		// into the scrap heap
		int viewType = lp.viewType;
		if (!shouldRecycleViewType(viewType)) {
			if (viewType != ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
				removeDetachedView(scrap, false);
			}
			return;
		}
		if (mViewTypeCount == 1) {
			dispatchFinishTemporaryDetach(scrap);
			mCurrentScrap.add(scrap);
		} else {
			dispatchFinishTemporaryDetach(scrap);
			mScrapViews[viewType].add(scrap);
		}
 
		if (mRecyclerListener != null) {
			mRecyclerListener.onMovedToScrapHeap(scrap);
		}
	}
 
	/**
	 * @return A view from the ScrapViews collection. These are unordered.
	 */
	View getScrapView(int position) {
		ArrayList<View> scrapViews;
		if (mViewTypeCount == 1) {
			scrapViews = mCurrentScrap;
			int size = scrapViews.size();
			if (size > 0) {
				return scrapViews.remove(size - 1);
			} else {
				return null;
			}
		} else {
			int whichScrap = mAdapter.getItemViewType(position);
			if (whichScrap >= 0 && whichScrap < mScrapViews.length) {
				scrapViews = mScrapViews[whichScrap];
				int size = scrapViews.size();
				if (size > 0) {
					return scrapViews.remove(size - 1);
				}
			}
		}
		return null;
	}
 
	public void setViewTypeCount(int viewTypeCount) {
		if (viewTypeCount < 1) {
			throw new IllegalArgumentException("Can't have a viewTypeCount < 1");
		}
		// noinspection unchecked
		ArrayList<View>[] scrapViews = new ArrayList[viewTypeCount];
		for (int i = 0; i < viewTypeCount; i++) {
			scrapViews[i] = new ArrayList<View>();
		}
		mViewTypeCount = viewTypeCount;
		mCurrentScrap = scrapViews[0];
		mScrapViews = scrapViews;
	}
 
}

```
里面的逻辑比较简单，这里只介绍几个比较重要的成员变量和方法
- mFirstActivePosition：第一个在mActiveViews存储的位置，其实可以这样理解，mActveViews表示的是当前可见的ListView的item对象，加入我们有100个数据，每次能够展示10个数据，那么第一次mFirstActivePosition的值就是0，如果我们滑动LstView，那么mFirstActivePosition可以变成1，2，3等数据
- mActiveViews：当前屏幕中可见的item集合，这个集合是从第一个可见的item开始到最后一个可见的item结束
- mScrapViews：从字面意思上来说是废弃的views，其实可以理解为当前不可见的view，需要通过adapter适配到convert view
- fillActiveView()：接受两个参数，第一个参数是可接收view的数量，第二参数表示ListView中第一个可见view的位置position，就像刚才所说，mActiveViews是用来存储当前可见的views，那么通过这个方法我们就会将指定item存储到mActiveViews数组中。
- getActiveView()：这个方法与fillActiveView()方法相呼应，用于从mActiveViews中获取数据，这个方法中会传递过来position参数，在方法中会将其转换成mActiveViews的下标。需要注意的是，一旦从mActiveViews中获取一个item对象，这个item对象就会从数组中移除，所以说mActiveViews不能重复利用
- addScrapView()：用于将一个废弃的view进行缓存，该方法接收一个view参数，当某个view被废除的时候(比如item滚出屏幕)通过该方法将view添加到mScrapViews集合中。
- setViewTypeCount()：我们在使用ListView时通过getViewTypeCout()来获取item的种类，那么在recyclerBin中，我们会为每种类型的数据单独启用一个RecyclerBin缓存机制，

# ListView机制
不管怎么说，ListView还是继承自View的，所以其绘制流程与View的绘制流程总是有千丝万缕的联系。
其实View的绘制主要就是三步走：onMeasure(),onLayout(),onDraw()，其中onMeasure和onDraw这两个方法我们可以不用考虑，主要是看onLayout方法。而onlayout方法主要是设置子view也就是item的位置。所以我们先来看一下ListView中的onLayout方法是如何实现的。可是我们惊奇的发现没有这方法，没事，不慌，ListView没有这个方法，我们就去他的父类AbsListView中查找
```
    /**
     * Subclasses should NOT override this method but
     *  {@link #layoutChildren()} instead.
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);

        mInLayout = true;

        final int childCount = getChildCount();
        if (changed) {
            for (int i = 0; i < childCount; i++) {
                getChildAt(i).forceLayout();
            }
            mRecycler.markChildrenDirty();
        }

        layoutChildren();

        mOverscrollMax = (b - t) / OVERSCROLL_LIMIT_DIVISOR;

        // TODO: Move somewhere sane. This doesn't belong in onLayout().
        if (mFastScroll != null) {
            mFastScroll.onItemCountChanged(getChildCount(), mItemCount);
        }
        mInLayout = false;
    }

```
在这个方法中，我们发现，主要做的事情就是ListView的大小和位置是否发生了改变，如果发生了改变，那么changed属性就会变成true，然后要求重新绘制所有的子View。除此之外，我们发现这里还调用了layoutChildren()方法，这个方法从字面意思上我们知道，他应该是排版子view的方法，所有我们需要进入到这个方法中看一下，但是这个方法在AbsListView中是空的，那么是不是就说明没有具体的实现呢？肯定不是，我们是在子类中实现的，这里我们看一下ListView中的layoutChildren方法
```
@Override
protected void layoutChildren() {
    final boolean blockLayoutRequests = mBlockLayoutRequests;
    if (!blockLayoutRequests) {
        mBlockLayoutRequests = true;
    } else {
        return;
    }
    try {
        super.layoutChildren();
        invalidate();
        if (mAdapter == null) {
            resetList();
            invokeOnItemScrollListener();
            return;
        }
        int childrenTop = mListPadding.top;
        int childrenBottom = getBottom() - getTop() - mListPadding.bottom;
        int childCount = getChildCount();
        int index = 0;
        int delta = 0;
        View sel;
        View oldSel = null;
        View oldFirst = null;
        View newSel = null;
        View focusLayoutRestoreView = null;
        // Remember stuff we will need down below
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            index = mNextSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                newSel = getChildAt(index);
            }
            break;
        case LAYOUT_FORCE_TOP:
        case LAYOUT_FORCE_BOTTOM:
        case LAYOUT_SPECIFIC:
        case LAYOUT_SYNC:
            break;
        case LAYOUT_MOVE_SELECTION:
        default:
            // Remember the previously selected view
            index = mSelectedPosition - mFirstPosition;
            if (index >= 0 && index < childCount) {
                oldSel = getChildAt(index);
            }
            // Remember the previous first child
            oldFirst = getChildAt(0);
            if (mNextSelectedPosition >= 0) {
                delta = mNextSelectedPosition - mSelectedPosition;
            }
            // Caution: newSel might be null
            newSel = getChildAt(index + delta);
        }
        boolean dataChanged = mDataChanged;
        if (dataChanged) {
            handleDataChanged();
        }
        // Handle the empty set by removing all views that are visible
        // and calling it a day
        if (mItemCount == 0) {
            resetList();
            invokeOnItemScrollListener();
            return;
        } else if (mItemCount != mAdapter.getCount()) {
            throw new IllegalStateException("The content of the adapter has changed but "
                    + "ListView did not receive a notification. Make sure the content of "
                    + "your adapter is not modified from a background thread, but only "
                    + "from the UI thread. [in ListView(" + getId() + ", " + getClass() 
                    + ") with Adapter(" + mAdapter.getClass() + ")]");
        }
        setSelectedPositionInt(mNextSelectedPosition);
        // Pull all children into the RecycleBin.
        // These views will be reused if possible
        final int firstPosition = mFirstPosition;
        final RecycleBin recycleBin = mRecycler;
        // reset the focus restoration
        View focusLayoutRestoreDirectChild = null;
        // Don't put header or footer views into the Recycler. Those are
        // already cached in mHeaderViews;
        if (dataChanged) {
            for (int i = 0; i < childCount; i++) {
                recycleBin.addScrapView(getChildAt(i));
                if (ViewDebug.TRACE_RECYCLER) {
                    ViewDebug.trace(getChildAt(i),
                            ViewDebug.RecyclerTraceType.MOVE_TO_SCRAP_HEAP, index, i);
                }
            }
        } else {
            recycleBin.fillActiveViews(childCount, firstPosition);
        }
        // take focus back to us temporarily to avoid the eventual
        // call to clear focus when removing the focused child below
        // from messing things up when ViewRoot assigns focus back
        // to someone else
        final View focusedChild = getFocusedChild();
        if (focusedChild != null) {
            // TODO: in some cases focusedChild.getParent() == null
            // we can remember the focused view to restore after relayout if the
            // data hasn't changed, or if the focused position is a header or footer
            if (!dataChanged || isDirectChildHeaderOrFooter(focusedChild)) {
                focusLayoutRestoreDirectChild = focusedChild;
                // remember the specific view that had focus
                focusLayoutRestoreView = findFocus();
                if (focusLayoutRestoreView != null) {
                    // tell it we are going to mess with it
                    focusLayoutRestoreView.onStartTemporaryDetach();
                }
            }
            requestFocus();
        }
        // Clear out old views
        detachAllViewsFromParent();
        switch (mLayoutMode) {
        case LAYOUT_SET_SELECTION:
            if (newSel != null) {
                sel = fillFromSelection(newSel.getTop(), childrenTop, childrenBottom);
            } else {
                sel = fillFromMiddle(childrenTop, childrenBottom);
            }
            break;
        case LAYOUT_SYNC:
            sel = fillSpecific(mSyncPosition, mSpecificTop);
            break;
        case LAYOUT_FORCE_BOTTOM:
            sel = fillUp(mItemCount - 1, childrenBottom);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_FORCE_TOP:
            mFirstPosition = 0;
            sel = fillFromTop(childrenTop);
            adjustViewsUpOrDown();
            break;
        case LAYOUT_SPECIFIC:
            sel = fillSpecific(reconcileSelectedPosition(), mSpecificTop);
            break;
        case LAYOUT_MOVE_SELECTION:
            sel = moveSelection(oldSel, newSel, delta, childrenTop, childrenBottom);
            break;
        default:
            if (childCount == 0) {
                if (!mStackFromBottom) {
                    final int position = lookForSelectablePosition(0, true);
                    setSelectedPositionInt(position);
                    sel = fillFromTop(childrenTop);
                } else {
                    final int position = lookForSelectablePosition(mItemCount - 1, false);
                    setSelectedPositionInt(position);
                    sel = fillUp(mItemCount - 1, childrenBottom);
                }
            } else {
                if (mSelectedPosition >= 0 && mSelectedPosition < mItemCount) {
                    sel = fillSpecific(mSelectedPosition,
                            oldSel == null ? childrenTop : oldSel.getTop());
                } else if (mFirstPosition < mItemCount) {
                    sel = fillSpecific(mFirstPosition,
                            oldFirst == null ? childrenTop : oldFirst.getTop());
                } else {
                    sel = fillSpecific(0, childrenTop);
                }
            }
            break;
        }
        // Flush any cached views that did not get reused above
        recycleBin.scrapActiveViews();
        if (sel != null) {
            // the current selected item should get focus if items
            // are focusable
            if (mItemsCanFocus && hasFocus() && !sel.hasFocus()) {
                final boolean focusWasTaken = (sel == focusLayoutRestoreDirectChild &&
                        focusLayoutRestoreView.requestFocus()) || sel.requestFocus();
                if (!focusWasTaken) {
                    // selected item didn't take focus, fine, but still want
                    // to make sure something else outside of the selected view
                    // has focus
                    final View focused = getFocusedChild();
                    if (focused != null) {
                        focused.clearFocus();
                    }
                    positionSelector(sel);
                } else {
                    sel.setSelected(false);
                    mSelectorRect.setEmpty();
                }
            } else {
                positionSelector(sel);
            }
            mSelectedTop = sel.getTop();
        } else {
            if (mTouchMode > TOUCH_MODE_DOWN && mTouchMode < TOUCH_MODE_SCROLL) {
                View child = getChildAt(mMotionPosition - mFirstPosition);
                if (child != null) positionSelector(child);
            } else {
                mSelectedTop = 0;
                mSelectorRect.setEmpty();
            }
            // even if there is not selected position, we may need to restore
            // focus (i.e. something focusable in touch mode)
            if (hasFocus() && focusLayoutRestoreView != null) {
                focusLayoutRestoreView.requestFocus();
            }
        }
        // tell focus view we are done mucking with it, if it is still in
        // our view hierarchy.
        if (focusLayoutRestoreView != null
                && focusLayoutRestoreView.getWindowToken() != null) {
            focusLayoutRestoreView.onFinishTemporaryDetach();
        }
        mLayoutMode = LAYOUT_NORMAL;
        mDataChanged = false;
        mNeedSync = false;
        setNextSelectedPositionInt(mSelectedPosition);
        updateScrollIndicators();
        if (mItemCount > 0) {
            checkSelectionChanged();
        }
        invokeOnItemScrollListener();
    } finally {
        if (!blockLayoutRequests) {
            mBlockLayoutRequests = false;
        }
    }
}

```
代码稍微多了一点，我们只看重点，首先我们可以确定的是，在第一次进入该方法的时候，我们的ListView中是没有数据的，为什么没有数据呢？我们可以这两理解，在我们初始化ListView之后，ListView中没有通过adapter设置数据，所以初始状态下ListView的数据是空的，而且刚才我们说到ListView通过adapter设置数据，其实也是通过adapter渲染数据item，所以这时候所对应的getChildCount也是0，后面代码也会运行RecyclerBin中的fillActiveViews方法，由于数据是空的，所以这行代码我们可以认为是没有做任何事情。紧接着我们会通过mLayoutMode设置布局模式，默认情况下是普通模式LAYOUT_NORMAL，然后我们会进入到fillFromTop方法，这个方法的实现如下所示
```
    /**
     * Fills the list from top to bottom, starting with mFirstPosition
     *
     * @param nextTop The location where the top of the first item should be
     *        drawn
     *
     * @return The view that is currently selected
     */
    private View fillFromTop(int nextTop) {
        mFirstPosition = Math.min(mFirstPosition, mSelectedPosition);
        mFirstPosition = Math.min(mFirstPosition, mItemCount - 1);
        if (mFirstPosition < 0) {
            mFirstPosition = 0;
        }
        return fillDown(mFirstPosition, nextTop);
    }

```
从注释中我们知道，他会从mFirstPosition往下的填充ListView，主要就是判断mFirstPosition的合法性，然后通过fillDown方法，填充。由于最终我们返回的是一个View对象，我们可以大概的知道，填充的具体操作，应该是在fillDown方法中执行的。fillDown方法代码如下所示
```
/**
 * Fills the list from pos down to the end of the list view.
 *
 * @param pos The first position to put in the list
 *
 * @param nextTop The location where the top of the item associated with pos
 *        should be drawn
 *
 * @return The view that is currently selected, if it happens to be in the
 *         range that we draw.
 */
private View fillDown(int pos, int nextTop) {
    View selectedView = null;
    int end = (getBottom() - getTop()) - mListPadding.bottom;
    while (nextTop < end && pos < mItemCount) {
        // is this the selected item?
        boolean selected = pos == mSelectedPosition;
        View child = makeAndAddView(pos, nextTop, true, mListPadding.left, selected);
        nextTop = child.getBottom() + mDividerHeight;
        if (selected) {
            selectedView = child;
        }
        pos++;
    }
    return selectedView;
}

```
在fillDown方法中，通过while循环执行重复逻辑，一开始nextTop的值是第一个子元素顶部距离整个ListView顶部的像素值，pos则是刚刚传入的mFirstPosition的值，而end是ListView底部减去顶部所得的像素值，mItemCount则是Adapter中的元素数量。因此一开始的情况下nextTop必定是小于end值的，并且pos也是小于mItemCount值的。那么每执行一次while循环，pos的值都会加1，并且nextTop也会增加，当nextTop大于等于end时，也就是子元素已经超出当前屏幕了，或者pos大于等于mItemCount时，也就是所有Adapter中的元素都被遍历结束了，就会跳出while循环。而在while循环中通过makeAndAddView方法，将view添加到相应的位置
```
/**
 * Obtain the view and add it to our list of children. The view can be made
 * fresh, converted from an unused view, or used as is if it was in the
 * recycle bin.
 *
 * @param position Logical position in the list
 * @param y Top or bottom edge of the view to add
 * @param flow If flow is true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @return View that was added
 */
private View makeAndAddView(int position, int y, boolean flow, int childrenLeft,
        boolean selected) {
    View child;
    if (!mDataChanged) {
        // Try to use an exsiting view for this position
        child = mRecycler.getActiveView(position);
        if (child != null) {
            // Found it -- we're using an existing child
            // This just needs to be positioned
            setupChild(child, position, y, flow, childrenLeft, selected, true);
            return child;
        }
    }
    // Make a new view for this position, or convert an unused view if possible
    child = obtainView(position, mIsScrap);
    // This needs to be positioned and measured
    setupChild(child, position, y, flow, childrenLeft, selected, mIsScrap[0]);
    return child;
}

```
在这个代码中我们通过RecyclerBin获取了一个activeView，并且通过setupChild方法将view设置到相应的位置，而此时view是null，所以会通过obtaionView方法设置一个view对象，保证child不为空
```
/**
 * Get a view and have it show the data associated with the specified
 * position. This is called when we have already discovered that the view is
 * not available for reuse in the recycle bin. The only choices left are
 * converting an old view or making a new one.
 * 
 * @param position
 *            The position to display
 * @param isScrap
 *            Array of at least 1 boolean, the first entry will become true
 *            if the returned view was taken from the scrap heap, false if
 *            otherwise.
 * 
 * @return A view displaying the data associated with the specified position
 */
View obtainView(int position, boolean[] isScrap) {
	isScrap[0] = false;
	View scrapView;
	scrapView = mRecycler.getScrapView(position);
	View child;
	if (scrapView != null) {
		child = mAdapter.getView(position, scrapView, this);
		if (child != scrapView) {
			mRecycler.addScrapView(scrapView);
			if (mCacheColorHint != 0) {
				child.setDrawingCacheBackgroundColor(mCacheColorHint);
			}
		} else {
			isScrap[0] = true;
			dispatchFinishTemporaryDetach(child);
		}
	} else {
		child = mAdapter.getView(position, null, this);
		if (mCacheColorHint != 0) {
			child.setDrawingCacheBackgroundColor(mCacheColorHint);
		}
	}
	return child;
}

```
在obtainView方法中我们会发现，通过getScrapView方法尝试获取一个废弃缓存中的view，由于获取不到，getScrapView会返回一个null，这时候在通过mAdapter.getView来获取一个view对象，而这个mAdapter就是与ListView相关联的adapter，而getView方法就是平时我们经常写在自定义Adapter中的getView方法，并且这个方法会返回一个View视图，此时我们的getView中会传递三个参数，对应的也就是adapter中的三个参数，其中scrapView就是convertView，如果这个对象为null，表示当前没有convertView可以利用，然后我们就会通过inflate方法动态加载一个布局，并且返回。由于不管getActiveView还是obtainView，最终都会执行setupChild方法，所以我们来看一下这个内容是如何实现的
```
/**
 * Add a view as a child and make sure it is measured (if necessary) and
 * positioned properly.
 *
 * @param child The view to add
 * @param position The position of this child
 * @param y The y position relative to which this view will be positioned
 * @param flowDown If true, align top edge to y. If false, align bottom
 *        edge to y.
 * @param childrenLeft Left edge where children should be positioned
 * @param selected Is this position selected?
 * @param recycled Has this view been pulled from the recycle bin? If so it
 *        does not need to be remeasured.
 */
private void setupChild(View child, int position, int y, boolean flowDown, int childrenLeft,
        boolean selected, boolean recycled) {
    final boolean isSelected = selected && shouldShowSelector();
    final boolean updateChildSelected = isSelected != child.isSelected();
    final int mode = mTouchMode;
    final boolean isPressed = mode > TOUCH_MODE_DOWN && mode < TOUCH_MODE_SCROLL &&
            mMotionPosition == position;
    final boolean updateChildPressed = isPressed != child.isPressed();
    final boolean needToMeasure = !recycled || updateChildSelected || child.isLayoutRequested();
    // Respect layout params that are already in the view. Otherwise make some up...
    // noinspection unchecked
    AbsListView.LayoutParams p = (AbsListView.LayoutParams) child.getLayoutParams();
    if (p == null) {
        p = new AbsListView.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.WRAP_CONTENT, 0);
    }
    p.viewType = mAdapter.getItemViewType(position);
    if ((recycled && !p.forceAdd) || (p.recycledHeaderFooter &&
            p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER)) {
        attachViewToParent(child, flowDown ? -1 : 0, p);
    } else {
        p.forceAdd = false;
        if (p.viewType == AdapterView.ITEM_VIEW_TYPE_HEADER_OR_FOOTER) {
            p.recycledHeaderFooter = true;
        }
        addViewInLayout(child, flowDown ? -1 : 0, p, true);
    }
    if (updateChildSelected) {
        child.setSelected(isSelected);
    }
    if (updateChildPressed) {
        child.setPressed(isPressed);
    }
    if (needToMeasure) {
        int childWidthSpec = ViewGroup.getChildMeasureSpec(mWidthMeasureSpec,
                mListPadding.left + mListPadding.right, p.width);
        int lpHeight = p.height;
        int childHeightSpec;
        if (lpHeight > 0) {
            childHeightSpec = MeasureSpec.makeMeasureSpec(lpHeight, MeasureSpec.EXACTLY);
        } else {
            childHeightSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
        }
        child.measure(childWidthSpec, childHeightSpec);
    } else {
        cleanupLayoutState(child);
    }
    final int w = child.getMeasuredWidth();
    final int h = child.getMeasuredHeight();
    final int childTop = flowDown ? y : y - h;
    if (needToMeasure) {
        final int childRight = childrenLeft + w;
        final int childBottom = childTop + h;
        child.layout(childrenLeft, childTop, childRight, childBottom);
    } else {
        child.offsetLeftAndRight(childrenLeft - child.getLeft());
        child.offsetTopAndBottom(childTop - child.getTop());
    }
    if (mCachingStarted && !child.isDrawingCacheEnabled()) {
        child.setDrawingCacheEnabled(true);
    }
}

```
在这个里面，我们将对应的子view--child通过addViewInLayout方法添加到ListView中，并且通过刚才在fillDown方法中的while循环将所有到的view添加进来。
到目前为止，第一个layout已经完成，那么还有第二次Layout


# RecyclerView

# 参考资料
[ListView工作原理完全解析](https://blog.csdn.net/guolin_blog/article/details/44996879)